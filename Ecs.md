In ECS, I would use two levels of health checks. At the container level, I configure an ECS Container Health Check to verify that the application process is actually healthy. At the load balancer level, I configure an ALB Target Group Health Check, for example /health or /actuator/health, which ensures that only healthy ECS tasks receive traffic. If a task is running but becomes unresponsive after several hours of inactivity, the ALB health check should detect it and stop routing traffic to that task, while ECS can replace the unhealthy task. I would also investigate application-level issues such as connection pool exhaustion, stale database connections, memory leaks, thread pool exhaustion, and ALB idle timeout rather than assuming it is only a health-check issue.”

🔥 Interviewer may ask:

“What is the difference between ECS health check and ALB health check?”

Answer:

“ECS Container Health Check checks the health of the container from the ECS perspective, while the ALB health check checks whether the application is reachable and healthy from the load balancer’s perspective. A container can be RUNNING but still fail the ALB health check.”
