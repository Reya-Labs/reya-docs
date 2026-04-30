# Rate Limits

To ensure fair usage and maintain optimal performance for all users, the Reya API implements rate limiting. The enforced rate limits are the following:

* **1,000 requests** per 60-second window per IP address/API key
* Rate limit counters reset at the end of each 60-second window

### Important Notes

* You can consume your entire quota at any point within the window (for example, all 1,000 requests in the first second)
* Once the limit is reached, additional requests will be rejected with a `429 Too Many Requests` status code until the current window expires
* Rate limits apply across all API endpoints collectively, not per endpoint

For most real-time data needs, we recommend using our [WebSocket API](/broken/pages/SL6x5f1Ngf6GtXYAOL10) instead of polling the REST API.

If your application requires higher rate limits, please contact the Reya team. The team will review your request and determine if higher limits can be granted based on your specific needs and usage patterns.
