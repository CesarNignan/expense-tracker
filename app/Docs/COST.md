# Cost Guide

What it costs to run this serverless Expense Tracker on AWS, which services charge you, which are free, and how to avoid surprise bills.

**Prices checked on:** 2026-11-09
**Region used:** Europe (London), `eu-west-2`
**Usage assumed:** a demo or learning project with a handful of users and a few hundred to a few thousand requests per month.

> Prices change and vary by region. Several official pricing pages only show US East (N. Virginia) examples, so numbers marked "not verified" should be confirmed with the [AWS Pricing Calculator](https://calculator.aws/) for your region before you rely on them.

## 1. Summary

- Without AWS WAF, the whole project should cost **close to $0 per month** at demo scale. Most services stay inside their free allowances.
- **AWS WAF is the only service with a fixed monthly charge.** A basic setup is roughly **$5 to $10 per month** even with zero visitors. It is optional and not needed for the app to work.
- If you skip WAF or delete it after your demo, the remaining usage costs are usually pennies.

## 2. Cost table

Sorted from most to least expensive at demo scale.

| Service | Role in this project | What you are charged for | Free allowance | Estimated monthly cost | Source |
| --- | --- | --- | --- | --- | --- |
| AWS WAF (optional) | Firewall in front of CloudFront | $5.00 per web ACL per month, $1.00 per rule or rule group per month, $0.60 per million requests | None found | **About $5 to $10** (1 web ACL plus 3 rules or rule groups is about $8) | [AWS WAF pricing](https://aws.amazon.com/waf/pricing) |
| Amazon CloudFront | Serves the website over HTTPS | Flat-rate plans or pay-as-you-go (data transfer and requests) | Free plan: $0 per month with 1M requests, 100 GB transfer and 5 GB S3 storage included | $0 on the Free plan. Pay-as-you-go rates for Europe: not verified | [CloudFront pricing](https://aws.amazon.com/cloudfront/pricing/) |
| Amazon S3 | Stores the HTML, CSS and JS files | Storage per GB-month, plus requests. Transfer from S3 to CloudFront is free | New accounts: up to $200 in credits (see section 7). London storage rate: not verified | Pennies (a few small files). Not verified | [S3 pricing](https://aws.amazon.com/s3/pricing/) |
| Amazon API Gateway (HTTP API) | Receives requests and checks the login token | $1.00 per million requests (first 300 million) | 1 million HTTP API calls per month for 12 months after sign-up | About $0.001 per 1,000 requests, and $0 inside the free tier | [API Gateway pricing](https://aws.amazon.com/api-gateway/pricing/) |
| Amazon DynamoDB (on-demand) | Stores the expenses | Per million write and read request units, plus storage above the free amount | 25 GB of storage per month. No free on-demand request units | Well under $0.01 at demo scale | [DynamoDB pricing](https://aws.amazon.com/dynamodb/pricing/?c=db&p=ft&z=3) |
| Amazon CloudWatch Logs | Stores Lambda logs | Log ingestion per GB, storage per GB | 5 GB per month across ingestion, storage and queries | $0 at demo scale | [CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/) |
| AWS Lambda | Runs the Python backend | $0.20 per million requests, plus compute per GB-second ($0.0000166667 for x86) | 1 million requests and 400,000 GB-seconds per month | $0 at demo scale | [Lambda pricing](https://aws.amazon.com/lambda/pricing/) |
| Amazon Cognito | Sign up, login and tokens | $0.015 per monthly active user above the free tier (Essentials tier) | First 10,000 monthly active users, which does not expire with the 12-month free tier | $0 | [Cognito pricing](https://aws.amazon.com/cognito/pricing/) |
| AWS IAM | Permissions for Lambda | No charge for IAM itself. Not verified in this check | n/a | $0 | [IAM](https://aws.amazon.com/iam/) |

## 3. Fixed costs (charged even when nobody uses the app)

**AWS WAF is the one to watch.**

- $5.00 per month for each web ACL.
- $1.00 per month for each rule you write and for each rule group or managed rule group you add.
- $0.60 per million requests inspected.
- Optional add-ons: Bot Control costs $10.00 per month per web ACL, and Fraud Control costs $10.00 per month for each of its two features. Do not enable these for a demo.

Everything else in this project is pay-per-use, so it costs nothing when the app sits idle.

## 4. Pay-per-use costs, explained

**AWS Lambda.** You pay per request and for compute time (memory multiplied by duration). The free tier covers 1 million requests and 400,000 GB-seconds each month. A demo with a few thousand calls of a few hundred milliseconds each is far below that. The page I read did not state whether this free tier is permanent, so check the current terms. Other regions can have different rates.

**API Gateway (HTTP API).** $1.00 per million requests. 1,000 requests cost about $0.001. The free tier of 1 million calls lasts 12 months from sign-up. I did not find any extra charge for the JWT authorizer on the pricing page, but it did not mention authorizers, so this is not confirmed.

**DynamoDB on-demand.** Example at US East rates: $0.625 per million write request units and $0.125 per million read request units. 1,000 writes cost about $0.0006 and 1,000 reads about $0.0001. Storage is free up to 25 GB per month, then $0.25 per GB-month (Standard). The free tier's 25 read and write capacity units apply to provisioned capacity, not on-demand. London rates were not shown on the page.

**Cognito.** The first 10,000 monthly active users are free and this does not expire after 12 months. After that, Essentials costs $0.015 per monthly active user. Essentials is the default tier for new user pools. The hosted login pages are part of the tier features, with no separate charge listed. Emails and SMS sent by Cognito can be billed separately through Amazon SES or SNS.

**S3 and CloudFront.** The site is a few small static files, so storage and requests cost very little. Data moving from S3 to CloudFront is free. CloudFront now has a flat-rate Free plan ($0 per month, 1M requests, 100 GB transfer, no overage charges) as well as pay-as-you-go pricing. I could not verify the pay-as-you-go European rates or any permanent free allowance on the page, so check which plan your distribution uses.

**CloudWatch Logs.** 5 GB per month is free. Beyond that, standard log ingestion is about $0.50 per GB (US East example). A small API produces a tiny amount of logs.

## 5. Optional extras

| Extra | Cost | Needed to run the app? |
| --- | --- | --- |
| AWS WAF | About $5 to $10 per month | No. It adds protection but the app works without it |
| WAF Bot Control | $10.00 per month plus request fees | No |
| WAF Fraud Control | $10.00 per month per feature | No |

## 6. Hidden costs and gotchas

- **WAF bills from the moment it is created**, even with zero traffic, and each extra rule adds $1.00 per month.
- **Free tiers expire.** The API Gateway free tier lasts 12 months. New accounts may also get time-limited credits (see section 7).
- **Log groups grow forever** unless you set a retention period. Set Lambda log retention to 7 or 14 days.
- **Forgotten resources.** A leftover WAF, user pool or CloudFront distribution keeps existing after you stop using the app.
- **Data transfer out.** CloudFront and API responses can incur transfer charges at high traffic. Not a concern at demo scale.
- **Wrong region.** Pricing and free tiers are applied per region. Create everything in one region, except WAF for CloudFront which must be in `us-east-1`.

## 7. How to avoid surprise bills

1. **Create a budget alert** first: Billing and Cost Management, then Budgets. Set it to something like $5 so you are warned early.
2. **Check Cost Explorer** weekly and group by service.
3. **New accounts:** AWS currently advertises up to $200 in free credits, with a free plan of 6 months and credits usable for 12 months. Check your account's Billing page to see what applies to you.
4. **Delete WAF after your demo** if you do not need it.
5. **Set log retention** on every CloudWatch log group.

## 8. Cleanup checklist (do it in this order)

1. **WAF:** remove the web ACL association from the CloudFront distribution, then delete the web ACL.
2. **CloudFront:** disable the distribution, wait until it is deployed, then delete it. Delete the Origin Access Control too.
3. **S3:** empty the bucket, then delete it.
4. **API Gateway:** delete the HTTP API (this removes its routes and authorizer).
5. **Lambda:** delete the function.
6. **CloudWatch:** delete the function's log group (`/aws/lambda/<function-name>`).
7. **DynamoDB:** delete the `Expenses` table.
8. **Cognito:** delete the user pool, which also removes the app clients and the hosted login domain.
9. **IAM:** delete the Lambda execution role and any policy you created for it.
10. **Billing:** check Cost Explorer a day or two later to confirm nothing is still charging, then remove the budget if you no longer need it.

## 9. Assumptions and disclaimer

- Usage assumed: a demo with a few users and a few thousand requests a month.
- The estimates above use the rates quoted on AWS's official pages on the date at the top. Some pages only show US East (N. Virginia) rates, so London prices may differ slightly.
- Not verified in this check: S3 London storage rate, CloudFront pay-as-you-go rates for Europe, whether the Lambda free tier is permanent, and IAM pricing.
- Run your own numbers with the [AWS Pricing Calculator](https://calculator.aws/).
