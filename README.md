# Gatling Workshop using Scala

# Introduction to Gatling

- Gatling is an open-source load-testing tool designed for web application testing using a **"load test as code"**
  approach.
- The supported build tools include **Maven, Gradle, and SBT** (Scala Build Tool).
- Makes use of **Akka and Netty frameworks** to provide efficient and scalable performance testing capabilities.
- Akka simplifies the construction of concurrent and distributed applications on the JVM.
- Netty is a client-server framework for the development of Java network applications such as protocol servers and
  clients
- Unlike JMeter, which starts up a thread for every virtual user that needs to run, Gatling uses a different structure
  that allows it to run **more than one user per thread**.
- The tool is known for its expressive **DSL (Domain Specific Language) written in Scala**.
- Additionally, Gatling features a script recorder with two modes: **HTTP proxy mode **and** HAR (HTTP Archive)
  converter mode**, facilitating the recording and conversion of scenarios for load-testing purposes.

# Prerequisite installations for the workshop

- Java Development Kit (JDK 8, JDK 11, or JDK 17 recommended)
- Scala (version 2.12 recommended)
- IntelliJ Idea or Visual Studio Code IDE (Scala Plugin to be installed)

# Gatling installation via different modes

**1. Standalone Installation**

- Go to [Open Source Load Testing | Gatling](https://gatling.io/open-source/) and download the open-source zip file.
- Extract the package and navigate to: 
**./gatling-charts-highcharts-bundle-3.10.3-bundle/gatling-charts-highcharts-bundle-3.10.3/bin**
- Open the terminal from the directory and enter **./gatling.sh** to run gatling.
- The gatling scripts to be simulated should be included in the directory: 
  **./gatling-charts-highcharts-bundle-3.10.3-bundle/gatling-charts-highcharts-bundle-3.10.3/user-files/simulations**
- Select the option to run the simulation locally and select the desired script.

**2. Installation using Maven**

- Ensure you have installed Maven, if not follow the steps given in this link: https://maven.apache.org/install.html
- Next, install the Scala Metals plugin inside VS Code or IntelliJ IDEA as per your preference.
- This plugin will enable a Scala language server and provide the typical features that you expect of an IDE.
- Next, you have to follow different steps based on the IDE that you use, for VS Code go to the Metals tabs and click on
  import build.
- In IntelliJ IDEA, setup the Scala SDK, and run the Engine.scala file to start the simulation.
- To run the script within VS Code, type **mvn gatling:test** in the terminal. If you want to run a specific test
  script, type in the following code instead:

        mvn test:gatling -DgatlingsimulationClass=&lt;package.simulationClassName>

- Running with runtime parameters:

        mvn test:gatling -DgatlingsimulationClass=&lt;package.simulationClassName> -DPARA1=VALUE1 -DPARA2=VALUE2

# Gatling Recorder

Gatling features a script recorder with HTTP proxy mode and HAR (HTTP Archive) converter mode.

- Navigate to: ./gatling-charts-highcharts-bundle-3.10.3-bundle/gatling-charts-highcharts-bundle-3.10.3/bin
- Open the terminal from the directory and enter ./recorder.sh to open the gatling recorder.

**1. HTTP proxy mode**

- Select the recorder mode as HTTP proxy and set the HTTPS mode as Self-signed Certificate and the port as 8080.
- Similarly, **set the proxy in the browser.**
- **NOTE:** For, Chrome the proxy has to be set for the entire system. Hence, consider an alternative like Firefox.
- Select the **script format as Scala** and change the class name if necessary.
- Click on **No static resources** to prevent the script from flooding with unwanted requests.
- Click on **Start**.

**2. HAR converter**

- Record the HAR file using developer tools. (remember to check preserve logs)
- Select the recorder mode as HAR converter and give the path to the HAR file.
- Select the **script format as Scala** and change the class name if necessary.
- Click on **No static resources** to prevent the script from flooding with unwanted requests.
- Click on **Start**.

# **Gatling Fundamental Sample Code using Scala**

```
package reqres.in

import io.gatling.core.Predef._ // Gatling's core DSL elements such as scenario, exec, http, setUp etc.
import io.gatling.http.Predef._ // Includes methods and constructs for defining HTTP requests, headers, checks etc.
import scala.concurrent.duration.DurationInt // For expressing durations in your code like milliseconds.

class GatlingFundamentals extends Simulation {

 // HTTP configuration
 val httpConf = http
   .baseUrl("https://reqres.in/api") // Base URL of the web application to load tested.
   .header("Accept", "application/json") // OR .acceptHeader("application/json")
   .contentTypeHeader("application/json")

 def getNumberOfResources() = {
   exec(http("Get the number of resources in the database")
     .get("/{resource}")
     .header("From", "TheFromHeader") // This sets an HTTP header named "From" with the value "TheFromHeader".
     .check(jsonPath("$.total").saveAs("totalResources")))
     .pause(3000.milliseconds)
 }

 def getResources() = {
   // Loop 2 times
   repeat(2) {
     exec(http("Fetches a resource list")
       .get("/{resource}")
       .queryParam("page", "1")
       .queryParam("per_page", "#{totalResources}")
       .check(status.is(200))
       .check(bodyString.saveAs("responseBody"))
       .check(regex(""""id":\d""")))
       // \d: Matches any digit character. \D: Matches any non-digit character (the opposite of \d).
       // The above text is raw string to avoid using double backslash triple quotes indicate raw string otherwise regular string.
       .exec { session => println(session("responseBody").as[String]); session }
       .exec { session => println(session); session }
   }
 }

 def getNumberOfUsers() = {
   exec(http("Get the number of users in the database")
     .get("/users")
     .check(jsonPath("$.total").saveAs("totalUsers")))
     .pause(3000.milliseconds)
 }
 def getSpecificUser() = {
   // another way to loop for a fixed number of times along with a counter that changes the value every loop from 0 to 4.
   repeat("#{totalUsers}", "counter") {
     exec(http(session => s"Get specific user with id: ${session("counter").as[Int] + 1}")
       .get(session => s"/users/${session("counter").as[Int] + 1}"))
   }
 }
 def register() = {
   exec(http("Creates a user")
     .post("/register")
     .body(StringBody("{\n    \"email\": \"eve.holt@reqres.in\",\n    \"password\": \"pistol\"\n}"))
     .check(jsonPath("$.token").saveAs("token"))
     .check(jsonPath("$.id").saveAs("id")))
 }
 def login() = {
   exec(http("Creates a user session")
     .post("/register")
     .body(StringBody("{\n    \"email\": \"eve.holt@reqres.in\",\n    \"password\": \"pistol\"\n}"))
     .check(jsonPath("$.token").is("#{token}")))
}
 def logout() = {
   exec(http("Ends a session")
     .post("/logout")
     .check(status.is(200), status.not(404), status.not(500), status.in(200 to 299)))
 }
 val scn = scenario("Testing")
   .exec(session => {
     session.set("totalResources", ""); session
   })
   .exec(getNumberOfResources())
   .exec(getResources())
   .exec(getNumberOfUsers())
   .exec(getSpecificUser())
   .repeat(3) {
     register()
     login()
     logout()
     pause(1, 20) // pause for 1 - 20 seconds
   }
setUp(
   scn.inject(atOnceUsers(1))
 ).protocols(httpConf)
}
```

# Gatling Feeders

**1. CSV Feeder**

- Save the csv file under the directory **./src/test/resources/data.**
- The below code snippet is used for defining a CSV feeder and using it in API:

```
val csvFeeder = csv("data/file.csv").circular
def getSpecificVideoGame() = {
    repeat(10) {
         feed(csvFeeder)
         .exec(http("Get specific video game")
         .get("videogames/${gameId}")
         .check(jsonPath("$.name").is("${gameName}"))
         .check(status.is(200)))
         .pause(1)
    }
}
```

- **csv()** is a Gatling method used to read data from a CSV file.
- The **circular** method is applied to the CSV feeder, making it circular.
- The other feeder strategies include queue, random, shuffle, circular etc.

**2. Custom Feeder**

- We need to create this template file. Inside the src > test > resources > bodies folder, create a new file called
  NewGameTemplate.json. Add in the following JSON:

```
{
 "id": ${gameId},
 "name": "${name}",
 "releaseDate": "${releaseDate}",
 "reviewScore": ${reviewScore},
 "category": "${category}",
 "rating": "${rating}"
}

```

- We then use the ElFileBody method to utilize the json template. Moreover, functions are used to create test data.

```
var idNumbers = (11 to 20).iterator
val rnd = new Random()
val now = LocalDate.now()
val pattern = DateTimeFormatter.ofPattern("yyyy-MM-dd")

def randomString(length: Int) = {
 rnd.alphanumeric.filter(_.isLetter).take(length).mkString
}

def getRandomDate(startDate: LocalDate, random: Random): String = {
 startDate.minusDays(random.nextInt(30)).format(pattern)
}

val customFeeder = Iterator.continually(Map(
 "gameId" -> idNumbers.next(),
 "name" -> ("Game-" + randomString(5)),
 "releaseDate" -> getRandomDate(now, rnd),
 "reviewScore" -> rnd.nextInt(100),
 "category" -> ("Category-" + randomString(6)),
 "rating" -> ("Rating-" + randomString(4))
))

def postNewGame() = {
 repeat(5) {
   feed(customFeeder)
     .exec(http("Post New Game")
       .post("videogames/")
         .body(ElFileBody("bodies/NewGameTemplate.json")).asJson
       .check(status.is(200)))
     .pause(1)
   }
 }
```

# Gatling Load Simulation

```
setUp(
   scn.inject(
     nothingFor(5 seconds),
     atOnceUsers(5),
     rampUsers(10) during (30)
   ).protocols(httpConf)
)
```

- **nothingFor(duration)** - Performs no execution for a fixed duration.
- **atOnceUsers(5)** - 5 virtual users will be injected immediately.
- **rampUsers(10) during (10 seconds)** - 10 virtual users will be ramped up linearly over a period of 10 seconds.

```
setUp(
 scn.inject(
   nothingFor(5 seconds),
   constantUsersPerSec(10) during (10 seconds),
   rampUsersPerSec(1) to (5) during (20 seconds)
 ).protocols(httpConf)
)
```

- **constantUsersPerSec(rate) during(duration)** - Injects users at a constant rate, defined in users per second, during
  a given duration. Users will be injected at regular intervals.
- **rampUsersPerSec(rate1) to (rate2) during(duration)** - Injects users from starting rate to target rate, defined in
  users per second, during a given duration. Users will be injected at regular intervals. Ramping will take place at
  regular intervals as per the rate.

```
val scn = scenario("Fixed Duration Load Simulation")
 .forever() {
   exec(getAllVideoGames())
     .pause(5)
     .exec(getSpecificGame())
     .pause(5)
     .exec(getAllVideoGames())
 }

 setUp(
 scn.inject(
   nothingFor(5 seconds),
   atOnceUsers(10),
   rampUsers(50) during (30 second)
 ).protocols(httpConf)
).maxDuration(1 minute)
```

- **.forever()** - loops the block of code infinitely.
- **.maxDuration(duration)** - sets a final limit to the load simulation irrespective of other parameters.

# Running Gatling with runtime parameters

```
private def getProperty(propertyName: String, defaultValue: String) = {
   Option(System.getenv(propertyName))
     .orElse(Option(System.getProperty(propertyName)))
     .getOrElse(defaultValue)
 }

 def userCount: Int = getProperty("USERS", "5").toInt
 def rampDuration: Int = getProperty("RAMP_DURATION", "10").toInt
 def testDuration: Int = getProperty("DURATION", "60").toInt
```

# Gatling Continuous Integration with Jenkins

- Beforehand, the source code should be pushed to a git repository.
- Follow the instructions in this [link](https://pkg.jenkins.io/debian-stable/) to install Jenkins.
- Navigate to the directory: /usr/share/java
- Run the command **java -jar jenkins.war --httpPort=9090**
- Take note of the administrator password given to you in the terminal.
- Go to the localhost:9090 server.
- Create a new admin and install the recommended plugins.
- Installing the gatling plugin: Manage Jenkins>Plugins>Available Plugins and install the gatling plugin.
- Click on + to create a new item>Freestyle Project
- **Configuring the project**

    - Enter in the git repository under source code management.
    - Next under Trigger check Poll SCM.
    - Then under Schedule put in 5 stars with whitespaces in between - \* \* \* \* \* to run the build every minute.
    - Under build steps select **“Invoke top-level Maven targets” **and enter the following command:

  **mvn test:gatling -DgatlingsimulationClass=&lt;package.simulationClassName>**

- **Running with runtime parameters.**
    - Check the box _“this project is parameterized”_.
    - Select string parameter.
    - Enter in the necessary parameters and values.

# Gatling Continuous Integration with Travis

- Go to [https://www.travis-ci.com/](https://www.travis-ci.com/) and sign up with your GitHub account.
- Authorize Travis CI to access your Git repository.
- Create and add .travis.yml file to the repository.
- Content of the YAML file would be like :

```
language: scala
sudo: false
script: "mvn gatling:test -Dgatling.simulationClass=finalSimulation.VideoGameFullTest"
```

# Gatling before and after block usage

```
class MySimulation extends Simulation {
 // Setup - executed once before the simulation starts
 before {
   // Code for setup tasks, e.g., initializing resources
   println("Setting up resources before the simulation starts...")
 }
 // HTTP Configuration
 val httpConf = http.baseUrl("https://example.com")
 // Scenario definition
 val scn = scenario("My Scenario")
   .exec(http("My Request").get("/"))
 // Injection profile
 setUp(
   scn.inject(atOnceUsers(1))
 ).protocols(httpConf)
 // Cleanup - executed once after the simulation completes
 after {
   // Code for cleanup tasks, e.g., releasing resources
   println("Cleaning up resources after the simulation completes...")
 }
}
```

# Gatling References and Documentations

- Google Forums Galting - [https://community.gatling.io/](https://community.gatling.io/)
- [https://www.james-willett.com/](https://www.james-willett.com/)
- https://gatling.io/docs/gatling/

# Gatling Certifications Gained

- https://academy.gatling.io/certificates/qq2f4dtidr
- https://www.udemy.com/certificate/UC-3cee6a43-61df-4c1e-8fd5-2af1aadf3b9a/

