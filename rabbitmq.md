### 几种工作模式

- 简单队列
- work模式：多个消费者取
  - 轮询分发
  - 公平分发
- 发布订阅模式fanout
  - 多个队列
  - 生产者向队列发送消息
- 路由模式
- 主题模式：
  - 字符串匹配



### 控制台的使用



### 整合SpringBoot

- 添加spring-boot-starter-amqp依赖

- application.yml
  - 配置IP和端口，用户名密码
- 配置类中添加org.springframework.amqp.core.Queue的**Bean**
- 简单模式和工作模式
  - 发送者使用**AmqpTemplate** 发送数据
  - 接收者
    - 在类上使用@RabbitListener注解标注要监听的队列
    - 在处理方法上使用@RabbitHandler注解标注接受方法，String类型参数表示消息内容



- Topic模式（一个交换机两个队列为例）
  - 发送者
    - 在配置类中声明多个队列及一个Topic类型的交换机
    - 绑定队列到交换机中并**指定某个队列的topic匹配规则**
    - 向交换机发送一条消息，并带上这条消息对应的topic，如果跟这个交换机所绑定的这些队列匹配的话，就会发送到匹配的队列中
  - 接收者
    - 与前面没啥区别



- Fanout发布订阅模式
  - 发送者
    - 绑定队列到Fanout类型的交换机，不需要指定匹配模式，这就是发布订阅模式
  - 接收者
    - 跟前面一样