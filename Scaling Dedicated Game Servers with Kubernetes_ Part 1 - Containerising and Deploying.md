Scaling Dedicated Game Servers with Kubernetes: Part 1 - Containerising and Deploying

For the past year and a half, I’ve been researching and building a variety of multiplayer games. Particularly those in the MMO or FPS genres, as they have some unique and very interesting requirements for communication protocols and scaling strategies. Due to my background with software containers and Kubernetes, I started to explore how they could be used to to manage and run dedicated game servers at scale. This became an area of exploration for me, and honestly I’ve been quite impressed with the results.

### Why Are You Doing This Thing?

While containers and Kubernetes are cool technologies, why would we want to run game servers on this platform?

- Game server scaling is *hard*, and often the work of proprietary software – software containers and Kubernetes should make it easier, with less coding.
- Containers give us a single deployable artifact that can be used to run game servers. This removes the need to install dependencies or configure machines during deployment, as well as greatly increases confidence that software will run the same on development and testing as it will in production.
- The combination of software containers and Kubernetes lets us build on top of a solid foundation for running essentially any type of software at scale – from deployment, health checking, log aggregation, scaling and more, with APIs to control these things at almost all levels.
- At its core, Kubernetes is really just a cluster management solution that works for almost any type of software. Running dedicated games at scale requires us to manage game server processes across a cluster of machines – so we can take advantage of the work already done in this area, and just tailor it to fit our specific need.
- Both of these projects are open source, and actively developed, so we can also take advantage of.any new features that are developed going forward

*> If you want to learn more about the intricacies of designing and developing communication and backends for FPS/MMO multiplayer games from the ground up I suggest starting with the articles on *> [*> Gaffer on Games*](http://gafferongames.com/)*>  or, if you prefer books, *> [*> The Development and Deployment of Multiplayer Games*](https://www.indiegogo.com/projects/development-deployment-of-multiplayer-games-vol1-book-online#/)*> .*

### Disclaimer

I will put one proviso on all of this. While I am aware of more than a few companies that are containerising their game servers and running them in production, I am not aware of any that are doing so on Kubernetes. I have heard of some that are *experimenting *with it, but don’t have any confirmed production uses at this stage. That being said, Kubernetes itself is being used by many large enterprises, and I feel this technology combination is solid, super interesting, and could potentially save game studios a lot of time. However, I am still working out where all the edges are. That being said, I will be happily sharing the results of all my research – and would love to hear from others on their experiences.

### Paddle Soccer

![](../_resources/6c6e09056307020b2a24de8d490ec331.png)
(Paddle Soccer in action. What a game!)

To test out my theories, I created a very simple Unity based game called *Paddle Soccer*, which is essentially exactly as described. It’s a two-player, online game in which each player is a paddle, and they play soccer, attempting to score goals against each other. It has a Unity client as well as a Unity dedicated server. It takes advantage of the [Unity High Level Networking API](https://docs.unity3d.com/Manual/UNetUsingHLAPI.html) to provide the game state synchronisation and UDP transport protocol between the servers and the client. If you are curious, all the code is available [on GitHub](https://github.com/markmandel/paddle-soccer) for your perusal.

It’s worth noting that this is a session-based game; i.e.. you play for a while, and then the game finishes and you go back to the lobby to play again, so we will be focusing on that kind of scaling as well as using that design to our advantage when deciding when to add or remove server instances. That being said, in theory these techniques would work with a MMO type game, with some adjustment.

### Paddle Soccer Architecture

Paddle Soccer uses a traditional overall architecture for session-based multiplayer games:

![Architecture diagram](../_resources/2f251c8d9cb0aa37e617c6ba2be1f2f4.gif)

1. Players connect to a *matchmaker service*, which pairs them together, using [Redis](https://redis.io/) to help facilitate this.

2. Once two players are joined for a *game session*, the matchmaker talks to the *game server manager*, to get it to provide a *game server* on our cluster of machines.

3. The *game server manager* creates a new instance of a *game server* that runs on one of the machines in the cluster

4. The *game server manager* also grabs the IP address and the port that the *game server* is running on, and passes that back the *matchmaker service*

5. The *matchmaker service* passes the IP and port back to the players’ clients

6. …and finally the players connect directly to the *game server* and can now start playing the game against each other

Since we don’t want to build this type of cluster management and game server orchestration ourselves, we can rely on the power and capabilities of containers and Kubernetes to handle as much of this work as possible.

### Containerising the Game Server

The first step in this process is putting the *game server* into a software container, so that Kubernetes can deploy it. Putting the game server inside a Docker container is essentially the same as containerising any other piece of software.

*> If this is not something you have done before, you’ll want to follow *> [*> Docker’s tutorial*](https://docs.docker.com/engine/getstarted/)*> , or if you like books, have a read of *> [*> The Docker Book: Containerization is the new virtualization*](https://www.amazon.com/gp/product/B00LRROTI4/ref=kinw_myk_ro_title)*> .*

Here is [the Dockerfile](https://github.com/markmandel/paddle-soccer/blob/master/server/game-server/Dockerfile) that is used to put the Unity dedicated game server in a container:

|     |     |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15 | FROM ubuntu:16.04<br>RUN useradd  -ms  /bin/bash unity<br>WORKDIR  /home/unity<br>COPY Server.tar.gz  .<br>RUN chown unity:unity Server.tar.gz<br>USER unity<br>RUN tar  --no-same-owner  -xf Server.tar.gz  &&  rm Server.tar.gz<br>ENTRYPOINT  ["./Server.x86_64",  "-logFile",  "/dev/stdout"] |

Because Docker runs as root by default, I like to create a new user and run all my processes inside a container under that account. Therefore, I’ve created a “unity” user for the game server and copied the game server into its home directory. As part of my build process, I create a tarball of my dedicated game server, and it’s been built such that it will run on a Linux operating system.

The only other interesting thing I do, is that when I set the ENTRYPOINT (the process to run when the container starts) I tell Unity to output the logs to /dev/stdout (standard out, i.e. display in the foreground), as that is where Docker and Kubernetes will take logs to aggregate from.

From here I am able to [build this image](https://docs.docker.com/engine/reference/commandline/build/) and push it to a [Docker registry](https://docs.docker.com/registry/), so that I can share and deploy this image to my Kubernetes cluster. I use Google Cloud Platform’s private [Container Registry](https://cloud.google.com/container-registry/) for this, so that I have a private and secure repository of my Docker images.

### Running the Game Server

For more traditional systems, Kubernetes provides several really useful constructs, including the ability to run multiple instances of an application across a cluster of machines and great tooling to load-balance between them.  However, for game servers, this is the direct opposite of what we want. Game servers usually maintain stateful data about players and the game in memory, and require very low latency connections to maintain the synchronicity of that state with game clients such that players do not notice a delay. Therefore, we need to have a direct connection to the game server without any intermediaries in the way adding latency, as every millisecond counts.

The first step is to run the game server. Each instance is not the same as the other, as they are stateful, so we can’t use a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) like we would for most stateless systems (such as web servers). Instead, we will lean on the most basic building blocks of deploying software on Kubernetes – the [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/).

A Pod is simply one or more containers that run together with some shared resources, such as an IP address and port space. In this particular instance, we will only have one container per Pod, so if it makes things easier to understand, just think of *Pod* as synonymous with *software container* for the duration of article.

### Connecting Directly to the Container

Normally, a container runs in its own network namespace and it isn’t directly connectable via the host without some work to forward the open ports inside the running container to the host. Running containers on Kubernetes is no different – usually you use a Kubernetes [Service](https://kubernetes.io/docs/concepts/services-networking/service/) as a load balancer to expose one or more backing containers. However, for game servers, that just won’t work, due to the low latency requirement for network traffic.

*> If you want to learn about the basics of deploying to Kubernetes, try the *> [*> interactive tutorials*](https://kubernetes.io/docs/tutorials/kubernetes-basics/)> .

Fortunately, Kubernetes allows Pods to use the host networking namespace directly by setting [*hostNetwork* to true when configuring the Pod](https://kubernetes.io/docs/resources-reference/v1.6/#podspec-v1-core). Since the container runs on the same kernel as the host, this gives us a direct network connection without additional latencies, and means we can connect directly to the IP of the machine the Pod is running on and connect directly to the running container.

While my example code makes [a direct API call against Kubernetes to create the Pod](https://github.com/markmandel/paddle-soccer/blob/master/server/sessions/kubernetes.go#L69), common practice is to keep your pod definitions in  YAML files that are sent to the Kubernetes cluster through the command line tool *kubectl*. Here’s an example of a YAML file that tells Kubernetes to create a Pod for the dedicated game server, so that we can discuss the finer details of what is going on:

|     |     |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15 | apiVersion: v1<br>kind: Pod<br>metadata:<br>  generateName: "game-"<br>spec:<br>  hostNetwork: true<br>  restartPolicy: Never<br>  containers:<br>    - name: soccer-server<br>      image: gcr.io/soccer/soccer-server:0.1<br>      env:<br>        - name: SESSION_NAME<br>          valueFrom:<br>            fieldRef:<br>              fieldPath: metadata.name |

Let’s break this down:
1. *kind
*Tell Kubernetes that we want a Pod!
2. *metadata > generateName*

Tell Kubernetes to generate a unique name for this Pod within the cluster, with the prefix “game-”

3. *spec > hostNetwork*

Since this is set to true, the Pod will run in the same network namespace as the host.

4. *spec > restartPolicy*

By default, [Kubernetes will restart a container if it falls over](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy). In this instance, we don’t want that to happen, as we have game state in memory, and if the server crashes, it’s very hard to restart back where the game was originally.

5. *spec > containers > image*

Tells Kubernetes which container image to deploy to the Pod. Here we are using the container image we created earlier for the dedicated game server.

6. *spec > containers > env > SESSION_NAME*

We are going to pass into the container the cluster-unique name for the Pod as an environment variable SESSION_NAME, as we will use it later. This is powered by the [Kubernetes Downward API](https://kubernetes.io/docs/tasks/configure-pod-container/environment-variable-expose-pod-information/).

If we deploy this YAML file to Kubernetes with the [kubectl command line tool](https://kubernetes.io/docs/user-guide/kubectl-overview/), and we know what port it is going to open, we can use the command line tools and/or the Kubernetes API to find the IP of the node in the Kubernetes cluster it is running on, and send that to the game client so it can connect directly!

Since we can also create a Pod via the Kubernetes API, Paddle Soccer has a game server management system called *sessions*, which has a */create* handler to create new instances of the game server on Kubernetes. When called, it will create a game server as a Pod with the above details. This can then be invoked via a matchmaking service whenever it has a need for a new game server to be started to allow two players to play a game!

We can also use the built-in Kubernetes API to [determine which node in the cluster the new Pod is on](https://github.com/markmandel/paddle-soccer/blob/master/server/sessions/kubernetes.go#L57), by looking it up from its generated Pod name. In turn, we can then look up the external IP of the node, and now we know what IP address to send to game clients.

This solves some problems for us already:

- We have a prebuilt solution for deploying a server to our cluster of machines through container images and Kubernetes.
- Kubernetes manages scheduling the game servers across the cluster, without us having to write our own [bin-packing](https://en.wikipedia.org/wiki/Bin_packing_problem) algorithm to optimise our resource usage.
- New versions of the game server can be deployed through standard Docker/Kubernetes mechanisms; we don’t need to write our own.
- We get all sorts of goodies for free – from log aggregation to performance monitoring and more.
- We don’t have to write much code (~500 LOC) to coordinate game servers across a cluster of machines.

### Port Management

Since we will likely have multiple dedicated game servers running on each of the nodes within our Kubernetes cluster, they will each need their own port to run on. Unfortunately, this isn’t something that Kubernetes will help us with, but solving this problem isn’t particularly difficult.

The first step is to decide on a range of ports that you want traffic to go through. This makes things easier for network rules for your cluster (if you don’t want to add/remove network rules on the fly), but also makes things easier for your players if they ever need to setup port forwarding or the like on their own networks.

To solve this problem, I tend to keep things as simple as possible: I pass the port range that could be used as two environment variables when creating my pod, and have the Unity dedicated server randomly select a value between that range, until it opens a socket successfully.

You can see the [Paddle Soccer Unity game server doing exactly this](https://github.com/markmandel/paddle-soccer/blob/master/unity/Assets/Scripts/Server/GameServer.cs#L112):

|     |     |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15 | public  static  void  Start(IUnityServer server)<br>{<br>    instance  =  new  GameServer(server);<br>    for  (var  i  =  0;  i  <  maxStartRetries;  i++)<br>    {<br>        *// select a random port in a range, and set it*<br>        instance.SelectPort();<br>        if  (instance.server.StartServer())<br>        {<br>            instance.Register();<br>            return;<br>        }<br>    }<br>    throw  new  Exception(string.Format("Could not find port"));<br>} |

Each call to [*SelectPort*](https://github.com/markmandel/paddle-soccer/blob/master/unity/Assets/Scripts/Server/GameServer.cs#L140) chooses a random port within a range, to be opened on *StartServer* invocation. [*StartServer*](https://github.com/markmandel/paddle-soccer/blob/master/unity/Assets/Scripts/Server/IUnityServer.cs#L26) will return *false* if it was unable to open a port and start the server.

You may also have noticed the call to [*instance.Register*](https://github.com/markmandel/paddle-soccer/blob/master/unity/Assets/Scripts/Server/GameServer.cs#L165). This is because Kubernetes doesn’t give us any way to introspect what port this container started on, so we’ll need to write our own. To that end, the Paddle Soccer game server manager has a simple */register* REST endpoint backed by Redis for storage that takes the Pod name that Kubernetes provides (which we pass through by environment variable), and stores the port the server started on. It also provides a */get* endpoint for looking up what port the game server started on.  This has been packaged along with the REST endpoints that create game servers, so we have [a single service](https://github.com/markmandel/paddle-soccer/tree/master/server/sessions) for managing game servers within Kubernetes.

Here is the dedicated [game server registering code](https://github.com/markmandel/paddle-soccer/blob/master/unity/Assets/Scripts/Server/GameServer.cs#L165):

|     |     |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11 | private  void  Register()<br>{<br>    var  session  =  new  Session<br>    {<br>        id  =  Environment.GetEnvironmentVariable("SESSION_NAME"),<br>        port  =  this.port<br>    };<br>    var  host  =  "http://sessions/register";<br>    server.PostHTTP(host,  JsonUtility.ToJson(session));<br>} |

You can see where the game server takes the environment variable SESSION_NAME with the cluster-unique Pod name and combines it with the port. This combinations is then sent as a JSON packet to the /*register* handler of the game server manager, *sessions*’  /*register* handler.

### Putting it All Together

If we combine this with the Paddle Soccer game clients, and a very simple matchmaker, we end up with the following:

![Kubernetes API step through ](../_resources/ee94ee3e973919c3c7c418d75f35e007.gif)

1. One player’s client connects to the matchmaker service, and it does nothing, since it needs two players to play

2. A second player’s client connects to the matchmaker service, and the matchmaker determines it needs a game server to connect these two players to, so it sends a request to the game server manager

3. The game server manager makes a call to the Kubernetes API to tell it to start a Pod in the cluster with the dedicated game server inside it

4. The dedicated game server starts

5. The dedicated game server registers itself with the game server manager, telling it what port it started on

6. The game server manager grabs the aforementioned port information, and the IP information for the Pod from Kubernetes and passes it back to the Matchmaker

7. The matchmaker passes the port and IP information back to the two player clients

8. The clients now connect directly to the dedicated game server, and play the game

Et Voilà! We have a multiplayer dedicated game running in our cluster!

In this example, a relatively small amount of custom code (~500 loc) was able to deploy, create, and manage game servers across a large cluster of machines by leveraging the power of software containers and Kubernetes.Honestly, it’s pretty awesome the power that containers and Kubernetes gives you!

This is but part one in the series however! In the next episode, we will look at how we can use APIs to scale up our Kubernetes cluster as demand for our game increases.

In the meantime, I welcome questions and comments here, or [reach out to me via Twitter](https://twitter.com/neurotic). You can see my [presentation at GDC](http://www.gdcvault.com/play/1024328/) this year on this topic as well as check out the code [in GitHub](https://github.com/markmandel/paddle-soccer), and still being actively worked on!