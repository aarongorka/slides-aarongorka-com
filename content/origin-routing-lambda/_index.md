---
title: "Origin Routing Lambda"
date: 2020-10-29T13:40:57+11:00
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

<style>
.reveal section img {
    border: none;
    box-shadow: none;
}
</style>

# Origin Routing Lambda

---

{{< table_of_contents >}} 

---

## Conflicting paths

1. `/some-path/some-page`: origin 1 (web app)
1. `/some-path/other-page`: origin 2 (CMS)

{{% fragment %}}No logical pattern to differentiate paths{{% /fragment %}}

---

## Requirements

  1. Business users can configure pages on origin 2 that display on origin 1 cache behaviours
  1. Code deployments are not required
  1. Ideally, business users do not have to explicitly/manually define which pages to display

{{% fragment %}}In other words; origin 2 pages should only display if the page is not present in origin 1.{{% /fragment %}}

---

## Some solutions?

  * Upload/embed a list of mappings of path:origin in the L@E
  * Use [origin failover with Origin Groups]

[origin failover with Origin Groups]: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/high_availability_origin_failover.html

---

## Origin Routing Lambda solution

![](/origin_routing_lambda_diagram.PNG)

---

## Code & Pipeline

  * Tightly coupled with CloudFront deployment
  * Same repo as CloudFront Infrastructure as Code (IaC)
  * L@E deployed just before CloudFront in pipeline

---

## Latency overhead

```python
r = requests.head(f"https://{hostname}{path}", headers=formatted_headers, params=params, timeout=1)
```

![](/origin_routing_lambda_latency_2.PNG)

---

## Cache

>If you want the function to execute for every request that CloudFront receives for the distribution, use the viewer request or viewer response events. **Origin request and origin response events occur only when a requested object isn't cached** in an edge location and CloudFront forwards a request to the origin.

https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-how-to-choose-event.html

---

## Testing

  * Smoke test, HTTP request to endpoints and assert response is coming from correct origin
  * Unit tests, "given x response from origin, assert y behaviour"

---

## Some exceptions

```python
inheritance_pages = [
    "/my-page",
    "/my-page/nested-page",
    "/other-page",
]
if path in inheritance_pages:
    logging.info(f"Path {path} is included in list of pages used in CMS for inheritance purposes, leaving origin as-is.")
    return request
```

---

## Troubleshooting L@E

  * Check response headers to differentiate origin
  * CloudWatch Logs (exceptions and stack traces): [search for `/aws/lambda/us-east-1` in _the region closest to your edge location_](https://ap-southeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-2#logsV2:log-groups$3FlogGroupNameFilter$3D$252Faws$252Flambda$252Fus-east-1)
  * Invalid function logs (view errors about improperly configured functions): [search for `/aws/cloudfront/LambdaEdge`](https://ap-southeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-2#logsV2:log-groups$3FlogGroupNameFilter$3D$252Faws$252Fcloudfront$252FLambdaEdge)
  * [CloudFront monitoring console (metrics)](https://console.aws.amazon.com/cloudfront/v2/home#/monitoring)
  * [Lambda console](https://console.aws.amazon.com/lambda/home) (note that for L@E, the logs & metrics are not populated here)
