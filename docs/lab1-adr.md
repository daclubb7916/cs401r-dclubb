## ADR-001: NorthStar Platform Foundation

### Status

Accepted

### Context

NorthStar Retail is building an AI platform that will be able to perform three different AI tasks. First, a Churn Prediction Model that will predict whether or not a customer will go silent over a 90-day period. Second, an Offer Generator that uses a RAG Model to help generate customer-specific offers that will improve the rate that offers are accepted. Third, an Agentic Model used for customer service that can handle routine issues which make up over 50% of customer service requests, and allow more complex issues to be handled by human customer service representatives.

An identity model is important because of the rules of least-privilege. Systems and individuals should only have access to the services and resources that is necessary to perform their job. This helps reduce security risks as well as reducing the likelihood of costly/damaging mistakes that could occur, and it should be defined as early as possible. A storage tier system is important early on because the very first steps in AI system development after defining a problem is making sure that the data available is sufficient to perform the desired AI tasks. Storage tier systems also can improve data pipelining capabilities.

### Decision

NorthStar will use a VPC within AWS Region us-east-1 that will contain an Availability Zone and a public and private subnet. An internet gateway attached to our VPC will route to the public subnet. The private subnet will contain a SageMaker Studio with start with an attached MLENgineer IAM role that will allow for users to access data necessary for development. The SageMaker domain and user profile attach to a security group that accepts traffic only from the VPC and permits outbound traffic.

NorthStar will use an S3 bucket within the region that has public access blocked. There are four tiers within the bucket, raw/, processed/, features/, and artifacts/. This bucket will contain the data instrumental to performing tasks such as Churn Prediction and Offer Generation. The prefixes establish interfaces for transformation, training, and deployment. Versioning protects overwritten data or models and public-access blocking protects regulated data.

### Consequences

#### What this makes easy

- The four named prefixes give each pipeline stage an explicit input and output location, improving the quality of the data for our churn prediction
- One MLEnginner role and user profile keeps the environment inexpensive while attributing activity to a named role.
- A `/16` VPC with a `/24` development subnet leaves substantial address space for additional private subnets, endpoints, and Availability Zones when the offer generations and agent services are introduced.

#### What this makes harder

- One public subnet in one Availability Zone cannot meet the customer service agent's needs by itself; an Availability Zone failure can remove the entire development network path.
- A single engineer role's combined permissions increase impact if the role is compromised.

#### What would cause you to revisit this decision

We would revisit this decision when production must scale beyond the current capacity or data, models, and systems must be protected across multiple Availability Zones. NorthStar's customer-service availability target would specifically require redundant private subnets and workloads in at least two Availability Zones rather than the current single-subnet design.

### Alternative Considered

We considered separate AWS accounts, VPCs, buckets, and IAM roles for the churn, offer, and service-agent systems. This credible production design reduces vulnerability, simplifies retention policies, and improves attribution. We rejected it now because the systems reuse customer and product data, no production traffic exists, and duplicating controls would consume credits before access patterns are known. The shared foundation does not prevent later separation.

### AWS Service Selection

- **Networking isolation model:** Amazon VPC supplies a dedicated CIDR, subnet, route table, internet gateway, and security group so Studio development has an explicit boundary that can later expand to multi-AZ private networking.
- **Storage design:** Amazon S3 provides durable, versioned, encrypted object storage whose four prefixes map directly to NorthStar's ingestion, transformation, feature, and model-artifact stages.
- **Identity model:** AWS IAM provides a SageMaker-trusted `MLEngineer` role with scoped S3 and ML permissions, establishing attributable workload access without exposing every data tier.
- **ML development environment:** Amazon SageMaker Studio provides managed notebooks, training, experiment tooling, and deployment integration for the churn, RAG, and agent workloads without maintaining development servers.
