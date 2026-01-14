# Overview

![](assets/Pasted_image_20260112154115.png)

Buff is an easy difficulty Windows machine that features an instance of Gym Management System 1.0. This is found to suffer from an unauthenticated remote code execution vulnerability. Enumeration of the internal network reveals a service running at port 8888. The installation file for this service can be found on disk, allowing us to debug it locally. We can perform port forwarding in order to make the service available and exploit it.
