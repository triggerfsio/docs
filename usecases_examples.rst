Why do I need triggerFS?
########################

TriggerFS is here to help you with the right tool. We don't want to invent anything new, we want to take already existing parts and put them together to build a system which can deliver the experience many people are looking for when:

- there is a new problem to be solved
- building distributed systems can actually be tricky
- suddenly automating things becomes relevant
- communication across boundaries is necessary but firewalls are in your way
- tasks need to be outsourced to other systems
- tasks need to be scheduled
- periodic tasks need to be run
- geo-replication is a topic
- a "serverless" design is taken into consideration
- lambda functions on own servers are needed
- fire&forget-style events could be needed
- wanting a central place where everything can be run (triggered) from

Benefits
########

triggerFS offers you one more way to access your server or services on that server.
If ssh is the door to get into your server, think about triggerFS as a door to access various, selected services on that server.

triggerFS' ``fs`` module also helps to enable M2M (machine-to-machine) communication on embedded devices. Since writing to files is a cheap operation, 
embedded devices with limited resources can do socket communication easily with triggerFS.

By inviting other triggerFS users into your team you can collaborate with each other and share resources within the team.
For example: Different universities can come together and share their servers to do data crunching or data analysis.

By joining other teams you can share your services team-wide with other users of that team. This way you can make your service accessable only for a set of users.

By making your service public you can offer your service to the whole world. People can then access your services either with the triggerFS ``client`` module or 
the HTTP/broker gateway via HTTP to make requests to your service.
