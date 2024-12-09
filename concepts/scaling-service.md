# Scaling Service

We’ve developed an innovative app called CraftBean that connects coffee enthusiasts with unique and artisanal coffee experiences worldwide.

At its core, CraftBean features the following components:

* **Consumer-facing apps**, unified in functionality and available on the following platforms:
  * **Web App**
  * **iOS App**
  * **Android App**
* **A robust stateless backend service**
* **A geographically distributed SQL database**

#### Key Features:

* Handpicked profiles of independent coffee roasters and cafés
* In-depth details about unique brewing styles and methods
* User-generated stories and reviews of standout coffee experiences
* Location-based discovery of hidden coffee gems
* Rich artisan profiles celebrating the craft of coffee making

#### Each café profile includes:

* Origins and philosophies behind the establishment
* Distinct brewing techniques
* Sourcing of specialty beans
* Profiles of the artisans behind the craft
* Reviews and shared experiences from users

## Initial Deployment (A single server)

<figure><img src="../.gitbook/assets/Screenshot 2024-12-09 at 18.46.06.png" alt=""><figcaption></figcaption></figure>

When we initially roll out a service, the user base and request volume are typically small. A single host can often handle this low level of traffic. To manage this, we configure our DNS to route all incoming requests to that single host.

## Expansion to Another Region

Moments later CraftBean gained more users from various region. During peak traffic times, our backend service handles thousands of requests per second, and our monitoring system has begun reporting 5XX status code errors caused by timeouts.

To reduce latency, services are typically hosted in regions close to their users. For a globally distributed user base, the service can be hosted in multiple regions, and **GeoDNS** is used to direct users to the nearest server by assigning multiple A records to a domain. GeoDNS determines the user’s location via their IP address and returns the corresponding server IP. If a data center becomes inaccessible, GeoDNS can provide an alternate IP address. These IP addresses can be cached at various levels, including the user’s ISP, operating system, and browser.

<figure><img src="../.gitbook/assets/Screenshot 2024-12-09 at 18.54.03.png" alt=""><figcaption></figcaption></figure>

## Caching Layer

High-traffic backend endpoints can be cached to improve performance. When there is a cache miss or for SQL databases/tables that aren't cached, the backend will retrieve the data directly from the database.

<figure><img src="../.gitbook/assets/Screenshot 2024-12-09 at 18.57.34.png" alt=""><figcaption></figcaption></figure>

## CDN for Static Files

Our browser apps previously hosted static content—files like JavaScript, CSS libraries, and certain images or videos—that remain the same for all users and are unaffected by user input. These files were included in our app’s source code repository, and users downloaded them from our Node.js service alongside the rest of the app. We decided to migrate this static content to a third-party Content Delivery Network (CDN). We provisioned sufficient capacity on the CDN, uploaded the files, updated our code to retrieve the files from the CDN URLs, and removed the static files from our source code repository.

A CDN stores copies of static files in multiple data centers worldwide, allowing users to download them with minimal latency, typically from the closest data center. However, if the nearest center is overloaded or experiencing issues, the files may be served from an alternative, faster location.

<figure><img src="../.gitbook/assets/Screenshot 2024-12-09 at 19.01.18.png" alt=""><figcaption></figcaption></figure>

## Deployment

### CI/CD

To enable rapid feature releases while reducing the risk of bugs, we implement continuous integration and continuous deployment (CI/CD) with unit and integration testing tools.

#### Release Strategy: Gradual Rollouts and Rollbacks

During deployment, we may release a new build incrementally across production hosts, such as 1%, 5%, 10%, up to 100%, while monitoring for issues. If problems arise—like undetected bugs, crashes, increased latency, memory leaks, or high resource usage—we can manually or automatically **roll back** to a stable version.

User churn is another factor to consider. Gradual rollouts allow us to observe how new builds impact user retention, as churn may result from performance issues or user dissatisfaction with changes.

For example, if a build increases latency beyond acceptable levels, we can mitigate this using caching and dynamic routing. A client experiencing a timeout might either retrieve data from its cache or retry the request, which could be routed to an older, stable build. Logging these events helps with troubleshooting.

We can configure our CD pipeline to divide production hosts into groups, gradually assigning builds to each group. If the cluster size changes, the pipeline can reassign or redeploy hosts as needed.

#### Experimentation

#### **Experimentation**

When introducing UX changes, new features, or design updates, we often roll them out gradually to an increasing percentage of users instead of all users at once. Unlike gradual rollouts, which focus on performance and user churn, **experimentation** aims to analyze how UX changes influence user behavior. Common approaches include **A/B testing** and **multivariate testing** (e.g., multi-armed bandit).

Experimentation also enables personalized user experiences.

A key difference from gradual rollouts is that experimentation uses dedicated tools (e.g., feature flags) to adjust the percentage of users exposed to specific builds, while gradual rollouts rely on CD tools to manage or revert builds based on detected issues.

Both **CD** and experimentation allow for short feedback cycles, enabling quick evaluation of deployments and features.

### Infrastructure as a Code

Our services are containerized using Docker, managed on host clusters with Kubernetes for scaling and load balancing, and configured using tools like Ansible or Terraform across multiple clusters.

### API gateway

When partnering with external companies, we may provide services like recommending products or services to users based on their in-app activity. To facilitate this, we develop an **API gateway layer** to expose specific APIs to external developers.

An **API gateway** acts as a reverse proxy that routes client requests to appropriate backend services. It centralizes common functionalities, reducing duplication across services, including:

* Authorization, authentication, and access control policies
* Logging, monitoring, and request-level alerting
* Rate limiting
* Billing
* Analytics

The gateway handles these functions, performs DNS lookups, and forwards the requests to the appropriate service hosts. It also interacts with supporting services like DNS, identity and access control, and rate-limiting configurations.

<figure><img src="../.gitbook/assets/Screenshot 2024-12-09 at 19.16.05.png" alt=""><figcaption></figcaption></figure>
