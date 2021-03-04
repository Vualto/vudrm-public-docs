# RELEASE NOTES 

* [December 2020](#december-2020)
* [November 2020](#november-2020)
* [September 2020](#september-2020)
* [July 2020](#july-2020)
* [May 2020](#may-2020)

## December 2020 

* Elasticsearch migration to improve reliablity.

* Various improvements to VUDRM Admin, including a new status page, a HTTPS redirect, and speed improvements to the dashboard. 

## November 2020

* Fairplay license server now accepts a more standard request.

* Improved internal logging.

* All license requests can now accept a VUDRM token in the request body, header, or as a query string parameter.

* VPN's and TOR networks can now be blocked by setting `block_vpn_and_tor` in a VUDRM token's policy.

* Widevine license server now returns `x-request-id` instead of `vualto-transaction-id` bringing it inline with web standards.

## September 2020

* Improvements to license server responses to aid in issue detection.

* New VUDRM cluster based in Paris has been released and is serving traffic.

* Improvement to internal apis to reduce length of license requests.

* `match_content_id` in token policy now support content encrypted with multiple keys from our CPIX API.

## July 2020

* New VUDRM cluster based in Oregon has been released and is serving traffic.

* Improved internal logging.

* Improved stats recorded.

* NGINX VOD Module support added to CPIX API.

* Harmonic support added to CPIX API.

* GEO based restrictions in can now be set in VUDRM token policy.

## May 2020

* New VUDRM cluster based in Bahrain has been released and is serving traffic.

* New endpoint for VUDRM CPIX API has been release to work with USP version 1.9.5.

* Infrastructure update to ensure latest security patches e.t.c are running on all VUDRM clusters.
