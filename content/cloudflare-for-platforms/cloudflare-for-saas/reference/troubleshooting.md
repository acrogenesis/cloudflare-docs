---
title: Troubleshooting
pcx_content_type: troubleshooting
weight: 3
meta:
  title: Troubleshooting Cloudflare for SaaS
---

# Troubleshooting Cloudflare for SaaS

## Rate limits

By default, you may issue up to 15 certificates per minute. Only successful submissions (POSTs that return 200) are counted towards your limit. If you exceed your limit, you will be prevented from issuing new certificates for 30 seconds.

If you require a higher rate limit, contact your Customer Success Manager.

---

## Purge cache

To remove specific files from Cloudflare’s cache, [purge the cache](/cache/how-to/purge-cache/purge-by-single-file/) while specifying one or more hosts.

---

## Resolution error 1016 (Origin DNS error) when accessing the custom hostname

Cloudflare returns a 1016 error when the custom hostname cannot be routed or proxied.

There are three main causes of error 1016:

1.  Custom Hostname ownership validation is not complete. To check validation status, run an API call to [search for a certificate by hostname](/cloudflare-for-platforms/cloudflare-for-saas/start/common-api-calls/) and check the verification error field: `"verification_errors": ["custom hostname does not CNAME to this zone."]`.
2.  Fallback Origin is not [correctly set](/cloudflare-for-platforms/cloudflare-for-saas/start/getting-started/#step-1--create-fallback-origin). Confirm that you have created a DNS record for the fallback origin and also set the fallback origin.
3.  A Wildcard Custom Hostname has been created, but the requested hostname is associated with a domain that exists in Cloudflare as a standalone zone. In this case, the [hostname priority](/ssl/reference/certificate-and-hostname-priority/#hostname-priority-cloudflare-for-saas) for the standalone zone will take precedence over the wildcard custom hostname. This behavior applies even if there is no DNS record for this standalone zone hostname.

In this scenario each hostname that needs to be served by the Cloudflare for SaaS parent zone needs to be added as an individual Custom Hostname.

{{<Aside type="note">}}

If you encounter other 1XXX errors, refer to [Troubleshooting Cloudflare 1XXX Errors](/support/troubleshooting/cloudflare-errors/troubleshooting-cloudflare-1xxx-errors/).

{{</Aside>}}

---

## Custom hostname in Moved status

To move a custom hostname back to an Active status, send a [PATCH request](/api/operations/custom-hostname-for-a-zone-edit-custom-hostname) to restart the hostname validation. A Custom Hostname in a Moved status is deleted after 7 days.

In some circumstances, custom hostnames can also enter a **Moved** state if your customer changes their DNS records pointing to your SaaS service. For more details, refer to [Remove custom hostnames](/cloudflare-for-platforms/cloudflare-for-saas/domain-support/remove-custom-hostnames/).

---


## CAA Errors

The `caa_error` in the status of a custom hostname means that the CAA records configured on the domain prevented the Certificate Authority to issue the certificate.

You can check which CAA records are configured on a domain using the `dig` command:
`dig CAA example.com`

You will need to ensure that the required CAA records for the selected Certificate Authority are configured.
For example, here are the records required to issue [Let's Encrypt](https://letsencrypt.org/docs/caa/) and [Google Trust Services](https://pki.goog/faq/#caa) certificates:

```
example.com CAA 0 issue "letsencrypt.org"
example.com CAA 0 issuewild "letsencrypt.org"

example.com CAA 0 issue "pki.goog; cansignhttpexchanges=yes"
example.com CAA 0 issuewild "pki.goog; cansignhttpexchanges=yes"
```

More details can be found on the [CAA records FAQ](/ssl/edge-certificates/troubleshooting/caa-records/).

## Custom hostname fails to verify because the zone is held

The [zone hold feature](/fundamentals/setup/account/account-security/zone-holds/) is a toggle that will prevent their zone from being activated on other Cloudflare account.
When the option `Also prevent subdomains` is enabled, this prevents the verification of custom hostnames for this domain. The custom hostname will remain in the `Blocked` status, with the following error message: `The hostname is associated with a held zone. Please contact the owner of this domain to have the hold removed.` In this case, the owner of the zone needs to [release the hold](/fundamentals/setup/account/account-security/zone-holds/#release-zone-holds) before the custom hostname can become activated.

