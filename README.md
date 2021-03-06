task-schedule
====================
基于zookeeper+quartz的分布式任务调度组件，非常小巧，使用简单，只需要引入jar包，不需要单独部署服务端。确保所有任务在集群中均匀分布，不重复，不遗漏的执行。
支持动态添加和删除任务。

功能概述
====================
1. 基于zookeeper+quartz的分布任务调度系统。
2. 确保每个任务在集群节点均匀分布，上不重复不遗漏的执行。
3. 单个任务节点故障时自动转移到其他任务节点继续执行。
4. 任务节点启动时必须保证zookeeper可用，任务节点运行期zookeeper集群不可用时任务节点保持可用前状态运行，zookeeper集群恢复正常运期。
5. 添加ip黑名单，过滤不需要执行任务的节点。
6. 管理页面，支持动态添加和删除任务。

任务持久化脚本
====================
	-- ----------------------------
	-- Table structure for `sys_task_schedule_job`
	-- ----------------------------
	DROP TABLE IF EXISTS `sys_task_schedule_job`;
	CREATE TABLE `sys_task_schedule_job` (
	  `id` int(11) NOT NULL AUTO_INCREMENT,
	  `job_name` varchar(255) DEFAULT NULL COMMENT '任务名称',
	  `job_status` varchar(1) DEFAULT NULL COMMENT '任务状态',
	  `job_group` varchar(255) NOT NULL COMMENT '任务组',
	  `cron_expression` varchar(255) DEFAULT NULL COMMENT '表达式',
	  `bean_class` varchar(255) DEFAULT NULL COMMENT '类路径',
	  `spring_id` varchar(255) DEFAULT NULL COMMENT 'springId',
	  `method_name` varchar(255) DEFAULT NULL COMMENT '方法名',
	  `is_concurrent` varchar(1) DEFAULT NULL COMMENT '是否同步',
	  `description` varchar(500) DEFAULT NULL COMMENT '描述',
	  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
	  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
	  `prj_name` varchar(50) DEFAULT NULL COMMENT '工程名',
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='定时任务配置表';

	-- ----------------------------
	-- Records of sys_task_schedule_job
	-- ----------------------------

	INSERT INTO `sys_task_schedule_job` VALUES ('12', 'host-monitor', '1', 'monitor', '0 0/10 * * * ?', 		'cn.com.citycloud.live.mgr.job', '', 'gogogo', '1', '任务测试', '2017-01-10 17:05:06', '2017-01-12 17:07:34', 'live-mgr');


	package cn.com.citycloud.live.mgr.job;
	/**
	 * 〈一句话功能简述〉<br> 
	 * 〈功能详细描述〉
	 *
	 * @author zhaoyi
	 * @see [相关类/方法]（可选）
	 * @since [产品/模块版本] （可选）
	 */
	public class JobTest {

	    public void gogogo(){
		System.out.println("如果觉得快乐你就拍拍手");
	    }
		
spring配置
====================
<!-- 扫描路径 -->
	    <context:component-scan base-package="cn.com.citycloud.frame.task"/>

	    <!-- 数据源注入 -->
	    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"/>
	    </bean>

	    <!-- 定时任务管理类 -->
	    <bean id="schedulerFactoryBean" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
		<property name="quartzProperties">
		    <props>
			<prop key="org.quartz.scheduler.skipUpdateCheck">true</prop> 
		    </props>
		</property>	
	    </bean>	

	    <!-- 分布式任务管理器-->
	    <bean id="zkScheduleManager" class="cn.com.citycloud.frame.task.ZKScheduleManager" init-method="init">
		<property name="zkConfig">
		    <map>
		       <entry key="zkConnectString" value="${zookeeper.address}" />
		       <entry key="rootPath" value="/task1.0/schedule" />
		       <entry key="zkSessionTimeout" value="60000" />
		       <entry key="userName" value="test" />
		       <entry key="password" value="test" />
		   </map>
		</property>
	    </bean>

API（实际运行ZK任务）
====================
1 动态添加任务

ConsoleManager.addScheduleTask(TaskDefine taskDefine);

2 动态删除任务

ConsoleManager.delScheduleTask(String targetBean, String targetMethod);

3 查询任务列表

ConsoleManager.queryScheduleTask();

API（持久化任务）
====================
cn.com.citycloud.frame.task.service.TaskScheduleJobService
持久化任务会在服务器启动的时候同步到实际运行ZK任务，用户在开发管理界面时，暂时需要考虑两者的同步问题。后期完善这个不足之处

task-schedule管理页面
====================
访问URL：项目名称/taskSchedule/home，如果servlet3.x以下，请手动配置web.xml文件
```
<servlet>
	    <servlet-name>TaskSchedule</servlet-name>
	    <servlet-class>cn.com.citycloud.frame.task.web.HomeServlet</servlet-class>
	</servlet>
	<servlet-mapping>
	    <servlet-name>TaskSchedule</servlet-name>
	    <url-pattern>/taskSchedule/*</url-pattern>
	</servlet-mapping>
```

关于
====================
改进于uncode-schedule，作者：冶卫军（ywj_316@qq.com,微信:yeweijun）

作者：赵毅（531559024@qq.com）
