# helloword

## 配置文件配置与引擎获取

```java
ProcessEngine processEngine = ProcessEngineConfiguration
        .createProcessEngineConfigurationFromResourceDefault()//读取activiti.cfg.xml方式
        .buildProcessEngine();
ManagementService managementService = processEngine.getManagementService();

Map<String, Long> tableCount = managementService.getTableCount();//获取表的数量
for(Map.Entry en : tableCount.entrySet()){
    logger.info("key:"+en.getKey()+" value:"+en.getValue());
}
```

activiti.cfg.xml:

```xml
<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
    <property name="jdbcUrl" value="jdbc:mysql://192.168.94.139:3306/activitidb" />
    <property name="jdbcDriver" value="com.mysql.jdbc.Driver" />
    <property name="jdbcUsername" value="root" />
    <property name="jdbcPassword" value="sa@123456" />
    <!--databaseSchemaUpdate 是否新建表结构-->
    <property name="databaseSchemaUpdate" value="true" />
    <property name="asyncExecutorActivate" value="false" />
</bean>
```

