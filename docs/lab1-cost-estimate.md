| Component | Monthly Estimate | Key Assumptions | One Optimization |
|---|---:|---|---|
| SageMaker Studio | $3.00 | 2 hrs/day at $0.05/hour | Only use the Jupyter Lab on workdays, save $1.00 a month |
| S3 storage | $0.23 | 10 GB at $0.023/GB |  |
| Internet Gateway | $0.01 | $0.01/GB data transfer | Keep S3 traffic within `us-east-1` |
| DynamoDB (state lock) | $0.00 | On-demand, near-zero reads |  |
| S3 state bucket | $0.00 | Minimal storage |  |
| **Total** | **$3.24** | | |
