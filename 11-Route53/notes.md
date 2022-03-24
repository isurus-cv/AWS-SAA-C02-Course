## Route 53

### Public Hosted Zones

A DNS database for a specific domain.

Public = hosted on R53 provided **public DNS servers**

AWS provides at least 4 hosted name servers

This is globally resilient (Multiple DNS servers)

This is created with domain registration via R53, can be created seperately.

There is a monthly fee for non R53 hosted zones.

Host DNS records.

Hosted Zones are what the DNS system refererences, it becomes Authoritative
for a domain and visible to the public internet.
![image](https://user-images.githubusercontent.com/88237437/159875334-5046f424-e0e5-4822-b3fa-0bfc8d2248f8.png)


![image](https://user-images.githubusercontent.com/88237437/159615159-898748f3-e254-4f4d-b8d2-832280e6740c.png)

### Private Hosted Zones

![image](https://user-images.githubusercontent.com/88237437/159615421-d9063143-8ba3-46a0-9320-3cb23e2fefb6.png)

![image](https://user-images.githubusercontent.com/88237437/159615586-1db46923-daf7-4870-ae30-dee03c79df53.png)

R53 Split View Hosted Zones
![image](https://user-images.githubusercontent.com/88237437/159615764-99c0f6d6-53d2-49af-92f7-610819a5c6e7.png)

The public internet cannot access "accounting" record since it is hosted in private zone. It can only be accessed by VPCs associated with R53

### R53 CNAME vs ALIAS
![image](https://user-images.githubusercontent.com/88237437/159616099-ff96ce84-ea5c-4682-8686-a2f91e2472da.png)

![image](https://user-images.githubusercontent.com/88237437/159616331-49b23cd6-7646-4707-815f-83ddf004a1ac.png)

### Simple Routing
![image](https://user-images.githubusercontent.com/88237437/159616580-64eb6c2c-0157-4b79-b199-72e83564581f.png)

### Route 53 Health Checks

Route checks will allow for periodic health checks on the servers.
If one of the servers has a bug, this will be removed from the list.

If the bug gets fixed, the health check will pass and the server will be
added back into a healthy state.

These are seperate from, but used by records on Route 53.

These are performed by a fleet of global health checkers. If you think
they are bots and block them, this could cause alarms.

Checks occur every 30 seconds by default. This can be increased to 10 seconds
for additional costs. These checks are per health checker. Since there are many
you will automatically get one every few seconds. The 10 second option will
complete multiple checks per second.

There could be one of three checks

- TCP checks: R53 tries to establish TCP with end point within 10 seconds
- HTTP/HTTPS: Same as TCP but within 4 seconds. The end point must respond
with a 200 or 300 status code within 3 seconds of checking.
- String matching: Same as above, the body must have a string within the first
5120 bytes. This is chosen by the user.

It will be deemed healthy or unhealthy.

There are three types of checks.

- Endpoint checks
- Cloudwatch alarms
- Checks of checks

![image](https://user-images.githubusercontent.com/88237437/159914378-f52b53f8-892f-4b1d-a8b0-c7cb0967b096.png)

### Route 53 Routing Policies

6 types

- Simple : Route traffic to a single resource. Client queries the resolver
which has one record. It will respond with 3 values and these get forwarded
back to the client. The client then picks one of the three at random.
This is a single record only. No health checks.

- Failover : Create two records of the same name and the same type. One
is set to be the primary and the other is the secondary. This is the same
as the simple policy except for the response. Route 53 knows the health of
both instances. As long as the primary is healthy, it will respond with
this one. If the health check with the primary fails, the backup will be
returned instead. This is set to impliment active - passive failover.

![image](https://user-images.githubusercontent.com/88237437/159914792-6f8c03b1-05ed-4214-b2ef-155468824599.png)


- Weighted : Create multiple records of the same name within the hosted zone.
For each of those records, you provide a weighted value. The total weight
is the same as the weight of all the records of the same name. If all of the
parts of the same name are healthy, it will distribute the load based
on the weight. If one of them fails its health check, it will be skipped over
and over again until a good one gets hit. This can be used for migration
to seperate servers.

- Latency-based : Multiple records in a hosted zone can be created with
the same name and same type. When a client request arrives, it knows which
region the request comes from. It knows the lowest latency and will respond
with the lowest latency.

- Geolocation : Focused to delivering results matching the query of your
customers. The record will first be matched based on the country if possible.
If this does not happen, the record will be checked based on the continent.
Finally, if nothing matches again it will respond with the default response.
This can be used for licensing rights. If overlapping regions,
the priority will always go to the most specific or smallest region. The US
will be chosen over the North America record.

- Multi-value : One record with one name and multiple values in this record.
These will be health checked and the unhealthy responses will automatically
be removed.
