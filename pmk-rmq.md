RabbitMQ pacemaker's resource scripts short analysis
==========================================

What we have?
------------------------------------------

So far we've got four different resource agents' scripts:

* [A very basic script](http://pkgs.fedoraproject.org/cgit/rpms/rabbitmq-server.git/tree/rabbitmq-server.ocf) supplied in our RabbitMQ RPM package (RHOS RMQ RPM)
* [A basic one](https://github.com/rabbitmq/rabbitmq-server/blob/master/scripts/rabbitmq-server.ocf) maintained by RabbitMQ upstream (RabbitMQ)
* [HA resource script](https://github.com/rabbitmq/rabbitmq-server/blob/master/scripts/rabbitmq-server-ha.ocf) recently upstreamed by Mirantis (RabbitMQ HA)
* Another one [script for RabbitMQ cluster](https://github.com/ClusterLabs/resource-agents/blob/master/heartbeat/rabbitmq-cluster) maintained by resource-agents'
  developers (Red Hat engineers)

Here is a short feature matrix:

|                 | RHOS RMQ RPM  | RabbitMQ | RabbitMQ HA | Resource Agents |
| --------------  | :-----------: | :------: | :---------: | :-------------: |
| PID for stop*1  | no            | yes      | yes*2       | yes             |
| Clustering      | no            | no       | yes         | yes             |
| Join cluster    | no            | no       | yes         | yes             |
| Repair cluster  | no            | no       | yes         | yes*3           |
| Boostrap cluster| no            | no       | yes         | yes             |
| Master/Slave    | no            | no       | yes*4       | no              |
| Reset node      | no            | no       | yes*5       | yes*6           |
| Easy to maintain| yes           | yes      | no*7        | yes*8           |
| Userbase        | unknown       | unknown  | Mirantis & Co | RedHat & Clients|
| Development pace| stalled       | stalled  | very active | maintenance mode|
| systemd usage*9 | no            | no       | no          | no              |
| LoCs            | < 400         | < 400    | > 1800*10   | < 400           |

1. Does it use PID-value from PID-file for stopping application?
2. It passes PID-file to rabbitmqctl. If can't be stopped, uses the value of a
   PID-file to kill (SIGTERM, or SIGKILL if can't terminate). If no or bad
   value, it matches by the beam process plus the rabbit node name, then kills.
3. Patch applied to src.rpm. Not yet upstreamed.
4. It is Master/Master (A/A) for the rabbit app. But there is a Master resource
   in a Pacemaker, which joins Slave resources to form a cluster.
5. Uses rabbitmqctl to reset node (in case of any issues). If can't reset, it
   stops node and wipes out all the data as well.
6. Stops node and wipes out all the data (does rm -rf, thus making settings not
   listed in /etc/rabbitmq/rabbitmq.config go away).
7. Extremely complex. Includes Erlang code snippets which loaded into RabbitMQ
   on the fly. Although it has UML flow charts for main events.
8. Sort of.
9. Startup notifications provided by systemd.
10. Excluding comments and whitelines.

Conclusion
------------------------------------------

RabbitMQ HA script is a clear winner in terms of feature-wise and development
activity. It gained a momentum recently and it looks like the entire
development is focused around it.

What is provided with RabbitMQ package should be simply trashed. It's an old
version of a basic script provided by RabbitMQ upstream and it doesn't offer
any benefits compared to the pacemaker systemd resource. Neither it provides
any clustering features (not even basic join).

We're using a script taken from resource-agents upstream. Unfortunately it
lacks behind "RabbitMQ HA" one. We should either consider joining RabbitMQ HA
development or upgrade this one seriously.
