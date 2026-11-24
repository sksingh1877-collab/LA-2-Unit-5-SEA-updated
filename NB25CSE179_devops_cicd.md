# Applying DevOps and CI/CD to an Online Shopping Platform

**Name:** Kshitiz Kumar Singh  
**USN:** NB25CSE179  
**Subject:** Software Engineering and Applications (Unit V)  
**Assigned Case Study:** Question 3 – DevOps Practices and CI/CD Pipelines  

---

## 1. Introduction

An online shopping platform requires regular updates to its product catalogue, payment system and customer-facing features. When development and operations teams work separately, releasing these updates can involve lengthy manual checks and configuration errors.

**DevOps** combines collaboration between these teams with automation across the software development lifecycle. Its objective is to make software releases faster, more consistent and easier to monitor.

### Continuous Integration, Delivery, and Deployment

At the heart of modern DevOps are three core automation practices:

- **Continuous Integration (CI):** Developers regularly merge or push their changes to a common repository such as GitHub. Each update initiates an automated build and testing process. Errors such as syntax issues, incorrect queries, or faulty logic can be detected quickly, before they become harder to resolve.
- **Continuous Delivery (CD):** After the code passes integration and automated tests, it is packaged into a deployable artifact or container and moved to a staging environment similar to production. The application remains ready for release, while a person typically approves its final deployment to production.
- **Continuous Deployment:** This practice goes a step further by eliminating the manual release approval. Every change that passes the required tests is automatically deployed to production.

---

## 2. Challenges in the Existing System

The organization in this case study operates an **online shopping application**, where a smooth customer experience and timely updates are important. The development team regularly introduces changes such as:
- Seasonal offers, product filters and promotional discount features.
- Corrections to shopping cart totals and checkout failures.
- Third-party payment gateway integrations (UPI, credit cards, net banking).

### Problems Associated with Manual Workflows

1. **Release Delays:** Manual verification takes considerable time. Testers have to check login, product searches, cart operations, and checkout flows across different browsers. As the application expands, regression testing can slow down releases and postpone important fixes.
2. **Deployment Errors:** Releasing software manually often requires copying files, setting up server configurations, and running scripts. Missing configuration values, incompatible dependencies, or migration mistakes can cause failures after deployment.
3. **Environment Differences ("It works on my machine"):** An application may run correctly on a developer's computer but fail on the production server because of differences in operating system packages, database versions, or runtime configuration.
4. **Poor Customer Experience:** Deployment problems can lead to unsuccessful payments, lost cart data, or server errors. Such interruptions affect sales and can reduce customer confidence in the shopping platform.

---

## 3. Designing an Automated CI/CD Workflow

To address these issues, the company can adopt an automated CI/CD pipeline built with tools such as Git, GitHub Actions or Jenkins, Docker, and Kubernetes.

### Pipeline Stages

The pipeline can be organized into the following stages:

1. **Source / Version Control:** Developers work on feature branches and push code to GitHub. Opening or merging a Pull Request into `main` automatically triggers the CI runner.
2. **Build Stage:** The automated server pulls the latest commit, downloads required dependencies, and compiles the shopping platform's services (e.g., auth, product catalog, payment service).
3. **Automated Testing Stage:** The build runs a battery of automated tests:
   - *Unit Tests:* Verify isolated functions (e.g., calculating item discounts, tax rounding, inventory decrement).
   - *Integration Tests:* Check interactions between modules (e.g., checking if the order service correctly communicates with the payment gateway mock and inventory database).
   - *End-to-End (E2E) / Smoke Tests:* Simulate a user logging in, adding a smartphone to the cart, applying a promo code, and completing checkout.
4. **Packaging & Containerization:** If all tests pass, Docker creates an immutable container image tagging the exact commit hash. This image contains the application runtime, dependencies, and configuration templates.
5. **Staging Deployment:** The image is automatically pushed to a container registry and deployed to a staging cluster that mirrors live production. Automated smoke checks run against staging to confirm health.
6. **Production Deployment:** For an e-commerce site, Continuous Delivery with rolling updates (or blue-green deployment) is recommended. Kubernetes swaps old pods with new pods incrementally. If health checks detect an error, the deployment can be halted and the previous stable version restored.
7. **Monitoring & Feedback:** Tools like Prometheus, Grafana, and error trackers log response latencies and server error rates. If checkout failures spike, the team receives instant alerts, and the metrics feed directly into the next sprint backlog.

---

## 4. CI/CD Pipeline Architecture Diagram

```
 +----------------------------------------------------------------------------------+
 |                            CONTINUOUS INTEGRATION (CI)                           |
 +----------------------------------------------------------------------------------+
        |
        v
  [ Developer Push ] 
        | (Git commit to GitHub)
        v
  +------------------+
  | 1. BUILD STAGE   | ---> Pull dependencies, compile frontend & backend microservices
  +------------------+
        | (Build Success)
        v
  +------------------+      +--------------------+      +----------------------+
  | 2. TEST STAGE    | ---> | Unit Tests (JUnit) | ---> | Integration Tests    |
  +------------------+      +--------------------+      +----------------------+
        |                                                          |
        | (All tests pass)                                         v
        |                                               +----------------------+
        |                                               | E2E Checkout Flow    |
        |                                               +----------------------+
        v
  +------------------+
  | 3. PACKAGE STAGE | ---> Build Docker image & push to container registry
  +------------------+
        |
 +----------------------------------------------------------------------------------+
 |                       CONTINUOUS DELIVERY / DEPLOYMENT (CD)                      |
 +----------------------------------------------------------------------------------+
        |
        v
  +------------------+
  | 4. STAGING       | ---> Deploy container to staging server & run smoke tests
  +------------------+
        |
        v
  [ Approval Gate ]  ---> Manual sign-off or automated criteria for live release
        |
        v
  +------------------+
  | 5. PRODUCTION    | ---> Zero-downtime rolling deployment (Kubernetes)
  +------------------+
        |
        v
  +------------------+
  | 6. MONITORING    | ---> Prometheus / Grafana tracking live latency & errors
  +------------------+
        |
        +------- Feedback Loop (Logs & alerts sent back to developers) -------> [ Next Fix ]
```

---

## 5. Detailed Explanation of Build, Test, and Deployment

### A. Build
Whenever a developer pushes a change, the CI system creates or uses a clean build environment. It installs the necessary dependencies, performs formatting and lint checks, and builds the storefront along with the backend order-processing components. 

*Shopping App Example:* If a developer modifies the checkout controller to support an additional UPI payment option, the build stage confirms that the code compiles without missing syntax or broken dependencies. If there is a compilation error, the build breaks immediately, alerting the developer before any code reaches QA.

### B. Test
Automated testing reduces the need for repetitive manual checks and generally includes the following test types:
- **Unit Testing:** Fast tests verifying individual algorithms (e.g., ensuring a "15% OFF" coupon caps out at the max discount amount).
- **Integration Testing:** Tests verifying API contracts (e.g., ensuring order creation triggers reservation of stock in the inventory database).
- **Security & Vulnerability Checks:** Scanners check third-party npm/pip packages for known vulnerabilities (CVEs) before releasing.

*Shopping App Example:* If an update to the payment module accidentally returns an undefined response for card expiration validation, automated tests can identify the defect before deployment.

### C. Deployment
Deployment moves the validated application version to live servers while aiming to avoid interruptions for shoppers. Docker containers and Kubernetes help achieve this by:
- The exact container tested in staging is what runs in production, eliminating "works on my machine" issues.
- **Blue-Green or Rolling Updates:** Instead of shutting down the store for maintenance, new application pods are started and warmed up. Once they pass initial health checks, router traffic seamlessly switches over to the new version. If configured health checks fail, the deployment can be rolled back to the previous stable container.

---

## 6. Advantages and Implementation Challenges

### Key Benefits

1. **Faster Time-to-Market & Frequent Updates:** Instead of saving up fixes for a stressful monthly release, features and bug fixes can be deployed several times a day safely.
2. **Early Defect Detection:** Bugs are discovered within minutes of a commit. Fixing an error while the code is fresh in the developer's mind takes significantly less time and costs far less than fixing a production crash during a holiday sale.
3. **High Reliability & Zero-Downtime Releases:** Automated test suites and rolling deployments prevent broken code from affecting shoppers, ensuring high cart conversion rates and system availability.

### Real-World Challenges & Limitations

1. **Initial Setup and Maintenance Overhead:** Designing resilient pipeline scripts, containerizing microservices, and configuring secret management requires significant initial engineering effort. Flaky tests that randomly fail also need continuous maintenance.
2. **Database Migrations:** While stateless application code can easily be rolled back, database schema changes (e.g., dropping or altering columns in user order tables) require backwards-compatible migration strategies so active transactions don't fail during rolling updates.

---

## 7. Conclusion

An automated CI/CD workflow allows an e-commerce application to evolve without depending entirely on manual release procedures. Continuous integration detects defects early, while staging and controlled production deployments reduce release risks. Combined with monitoring and feedback, these practices help maintain a dependable shopping experience and support frequent improvements.

---

### References
- Sommerville, Ian. *Software Engineering*, 9th/10th Edition (DevOps and Continuous Integration chapters).
- Lab Manual Unit V: Software Applications and Emerging Trends.
