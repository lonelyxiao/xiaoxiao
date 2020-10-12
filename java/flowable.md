# 写在前面

本笔记皆以spring为基础

# 常用组件

```java
//流程存储组件
@Autowired
private RepositoryService repositoryService;
//运行时服务组件
@Autowired
private RuntimeService runtimeService;
//流程任务组件
@Autowired
private TaskService taskService;
```

# 流程demo

## 流程定义

- xml方式定义
- BpmnModel方式

```java
BpmnModel bpmnModel=new BpmnModel();
//开始节点的属性
StartEvent startEvent=new StartEvent();
startEvent.setId("start-test");
startEvent.setName("start-test-name");

//普通的UserTask节点
UserTask userTask=new UserTask();
userTask.setId("userTask-test");
userTask.setName("userTask-test-name");


//结束节点属性
EndEvent endEvent=new EndEvent();
endEvent.setId("endEvent-test");
endEvent.setName("endEvent-test-name");

//连线信息
List<SequenceFlow> sequenceFlows=new ArrayList<SequenceFlow>();
//开始节点与usertask连接
SequenceFlow s1=new SequenceFlow();
s1.setId("start-test-line");
s1.setName("start-test-line-name");
s1.setSourceRef("start-test");
s1.setTargetRef("userTask-test");
sequenceFlows.add(s1);

//usertask与结束连接
List<SequenceFlow> toEnd=new ArrayList<SequenceFlow>();

SequenceFlow s2 = new SequenceFlow();
s2.setId("end-test-line");
s2.setName("end-test-line-name");
s2.setSourceRef("userTask-test");
s2.setTargetRef("endEvent-test");
toEnd.add(s2);

//设置开始节点出去的线
startEvent.setOutgoingFlows(sequenceFlows);
userTask.setOutgoingFlows(toEnd);

//设置进来的线
userTask.setIncomingFlows(sequenceFlows);
endEvent.setIncomingFlows(toEnd);

//Process对象
Process process=new Process();
process.setId(processId);
process.setName(processId);
process.addFlowElement(startEvent);
process.addFlowElement(s1);
process.addFlowElement(userTask);
process.addFlowElement(s2);
process.addFlowElement(endEvent);
bpmnModel.addProcess(process);
```

## 验证流程

```java
//查看流程绘制情况
BpmnXMLConverter bpmnXMLConverter=new BpmnXMLConverter();

byte[] convertToXML = bpmnXMLConverter.convertToXML(bpmnModel);

String bytes=new String(convertToXML);

System.out.println(bytes);


//验证bpmnModel 是否是正确的bpmn xml文件

ProcessValidatorFactory processValidatorFactory=new ProcessValidatorFactory();

ProcessValidator defaultProcessValidator = processValidatorFactory.createDefaultProcessValidator();

//验证失败信息的封装ValidationError
List<ValidationError> validate = defaultProcessValidator.validate(bpmnModel);
//如果size为0说明，bpmnmodel正确，大于0,说明自定义的bpmnmodel是错误的，不可以使用的。
System.out.println(validate.size());
```

## 部署流程

```java
//部署流程
Deployment deploy = repositoryService.createDeployment()
        .name("测试流程")
        .key(processId)
        .addBpmnModel("bpmnModel.bpmn", bpmnModel).deploy();

System.out.println(deploy.toString());
Deployment deployment = repositoryService.createDeploymentQuery().processDefinitionKey(processId).singleResult();
if (deployment == null){
    System.out.println("没有该流程");
}
```

## 启动流程

```java
runtimeService.startProcessInstanceByKey(processId);
```

## 提交节点任务

```java
 //查询任务
Task task = taskService.createTaskQuery().singleResult();
System.out.println("当前任务：" + task.getName());
taskService.complete(task.getId());
```

# 流程流转

## 获取当前流程实例的任务

当流程启动以后，流程会流转的下一个节点，所以此处list只有一个task，就是开始节点的下一个节点

```java
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(processId);
List<Task> list = taskService.createTaskQuery().processInstanceId(processInstance.getId()).list();
System.out.println(list.toString());
```