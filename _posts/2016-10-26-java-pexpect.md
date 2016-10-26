---
author: Fish
layout: post
title: Java 登录 Linux 并执行命令
categories: Python
tags: algorithm acm
---

测试工程中验证数据表, 为了省事, 直接登录到 Linux 执行命令 <code>mysql -u -p xx -e 'select * from xx'</code> 获取输出的内容.

```java
package com.alipay.liuqi.common.utils;

import com.jcraft.jsch.Channel;
import com.jcraft.jsch.ChannelExec;
import com.jcraft.jsch.JSch;
import com.jcraft.jsch.Session;
import org.apache.commons.lang.StringUtils;
import org.apache.log4j.Logger;

import java.io.InputStream;
import java.util.HashMap;
import java.util.Properties;

/**
 * DB 工具校验类
 * Created by fish on 25/10/16  16:15 with IntelliJ IDEA.
 */
public class DBCheckUtil {

    private static String HOST = ConfigUtil.getValue("SQL_SERVER");
    private static String USERNAME = ConfigUtil.getValue("SQL_SERVER_USERNAME");
    private static String PASSWORD = ConfigUtil.getValue("SQL_SERVER_PASSWD");
    private static String SQL_SELECT_TEMPLATE = ConfigUtil.getValue("SQL_SELECT_TEMPLATE");
    private static int PORT = 22;
    private static Session session = null;
    private static Channel channel = null;
    private Logger logger = Logger.getLogger(DBCheckUtil.class);


    /**
     * 校验数据表元素
     * <p>
     * conditon 格式: camp_id=123&&iprole_id=456
     * expectValue 格式: name=活动测试&&relation=exclusive
     *
     * @param tableName   数据表名
     * @param condition   查询条件
     * @param expectValue 期望条件
     * @return boolean    True || False
     */
    public boolean checkTable(String tableName, String condition, String expectValue) {

        // 解析condition, 拼装sql
        String[] conditionList = condition.split("&&");
        StringBuilder sqlCondition = new StringBuilder();
        for (String conditionElement : conditionList) {
            String[] kv = conditionElement.split("=");
            String key = kv[0];
            String value = kv[1];
            // 不直接组合字符串, 是为了 camp_id=123, 否则就要写成 camp_id='123'
            sqlCondition.append(String.format("%s='%s' ", key, value));
        }
        String executeSQL = String.format(SQL_SELECT_TEMPLATE, tableName, sqlCondition.toString());
        // 解析期望值, 进行校验
        HashMap<String, String> executeResult = getSQLExecuteResult(executeSQL);
        if (null != executeResult && executeResult.isEmpty()) {
            return false;
        }
        try {
            String[] expectValueList = expectValue.split("&&");
            for (String expectValueElement : expectValueList) {
                String[] kv = expectValueElement.split("=");
                String key = kv[0];
                String value = kv[1];
                // 如果期望值与查询出的值不相等, 直接返回false
                if (!StringUtils.equals(executeResult.get(key), value)) {
                    return false;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    /**
     * SQL 执行结果
     * <p>
     * 支持单行结果返回, 如果多行结果, 只取了第一行
     *
     * @param sql 查询语句
     * @return 查询结果 k-v
     */
    private HashMap<String, String> getSQLExecuteResult(String sql) {
        HashMap<String, String> resultMap = new HashMap<>();
        // 拼装 Linux 命令
        String command = String.format(ConfigUtil.getValue("SQL_SIT_COMMAND"), sql);
        String stringResult = executeCommand(command);

        // 获取结果返回
        String[] ret = stringResult.split("\n");
        if (ret.length < 2) {
            logger.error("No content output! Please check sql.");
            return null;
        }
        // 将列名和内容组装到 hashmap 中
        String[] columnName = ret[0].split("\t");
        String[] content = ret[1].split("\t");
        for (int i = 0; i < columnName.length; i++) {
            resultMap.put(columnName[i], content[i]);
        }
        logger.debug(resultMap);
        return resultMap;
    }

    /**
     * 初始化连接 Session 以及 channel
     *
     * @param host     服务器域名
     * @param username 登录名
     * @param passwd   登录密码
     */
    private void initSSH(String host, String username, String passwd) {
        try {
            logger.info("Start logging on server: " + host);

            //增加配置, 防止出现 UnknownHostKey Error
            Properties properties = new Properties();
            properties.put("StrictHostKeyChecking", "no");
            JSch jSch = new JSch();
            session = jSch.getSession(username, host, PORT);
            session.setPassword(passwd);
            session.setConfig(properties);
            session.connect();
            channel = session.openChannel("exec");
            logger.info(String.format("Logging in server %s success!", host));
        } catch (Exception e) {
            e.printStackTrace();
            logger.error(String.format("Log in server %s error", host));
        }
    }

    /**
     * 执行 Linux 命令
     *
     * @param command 待执行命令
     * @return String 执行结果
     */
    private String executeCommand(String command) {

//        // init Session
//        if (null == session || null == channel) {
//            initSSH(HOST, USERNAME, PASSWORD);
//        }
        initSSH(HOST, USERNAME, PASSWORD);
        byte[] tmp = new byte[1024];
        // 存放执行结果
        StringBuilder strBuffer = new StringBuilder();
        // 强制转换成 exec 类型的 channel
        ChannelExec channelExec = (ChannelExec) (channel);
        try {
            logger.info(String.format("Start executing command %s", command));
            InputStream inputStream = channelExec.getInputStream();
            InputStream errorStream = channelExec.getErrStream();
            // 执行 linux 命令
            channelExec.setCommand(command);
            channelExec.connect();
            // 获取执行结果
            while (true) {
                // Error output
                while (errorStream.available() > 0) {
                    int i = errorStream.read(tmp, 0, 1024);
                    if (i < 0) {
                        break;
                    }
                    strBuffer.append(new String(tmp, 0, i));
                    logger.error("Executing command ： " + command + " error:" + strBuffer.toString());
                }
                // Standard Output
                while (inputStream.available() > 0) {
                    int i = inputStream.read(tmp, 0, 1024);
                    if (i < 0) {
                        break;
                    }
                    strBuffer.append(new String(tmp, 0, i));
                }
                if (channel.isClosed()) {
                    break;
                }
                Thread.sleep(100);
            }
        } catch (Exception e) {
            e.printStackTrace();
            logger.error(String.format("Executing command %s on server %s", command, HOST));
        } finally {
            // 关闭 session 以及 channel 连接
            logger.debug("Close connection from " + HOST);
            if (null != channel) {
                channel.disconnect();
            }
            if (null != session) {
                session.disconnect();
            }
        }
        String executeResult = strBuffer.toString();
        logger.debug("Execute result: " + executeResult);
        return executeResult;
    }
}
```
