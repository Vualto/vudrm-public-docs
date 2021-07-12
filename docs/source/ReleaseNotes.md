# RELEASE NOTES 

* [July 2021](#july-2021)
* [June 2021](#june-2021)
* [May 2021](#may-2021)
* [April 2021](#april-2021)
* [March 2021](#march-2021)
* [February 2021](#february-2021)
* [January 2021](#january-2021)
* [December 2020](#december-2020)
* [November 2020](#november-2020)
* [September 2020](#september-2020)
* [July 2020](#july-2020)
* [May 2020](#may-2020)

## July 2021

* Updates to O/S on most services.

* Multikey support for all DRM providers added to CPIX API. Our documentation has been updated to reflect this.

* Internal improvements to the Widevine license server. Added license renewal support.

* Various improvements to VUDRM Admin, including a new public demo page.

* Internal improvements for our Stats API logging.

* Beta release for VUDRM Token v2.

## June 2021

* New VUDRM cluster based in Milan has been released and is serving traffic for the duration of the Euro 2020 Championship.

* Various improvements to VUDRM Admin, including the ability for the client to edit default VUDRM Token policy values.

## May 2021

* Various improvements to VUDRM Admin, including the integration of the JW Player client for testing purposes and Health page amends.

## April 2021

* Improvements for our Widevine logging. Logs that show the request start and completion now appear in Kibana.

* Bug fixes in the VUDRM Admin site, including a client cloning issue displaying incorrect errors and the user sometimes being redirected to the Login page incorrectly.

## March 2021

* Improvements to the AES license server. Now responds with a 4xx status code when the call to KP API fails.

* Improvements have been made to our Documentation. All documentation now follows a unified template throughout.

* Bug fix in the VUDRM Admin site where the Configuration page would sometimes break the service for certain clients.

* Improvements to the Widevine license server. Now forwards the Referer to the Stats API.

* Internal improvements to Kibana. Referer and License URL are displayed in Kibana.

## February 2021

* Improvements to our cluster infrastructure. Pods will automatically recycle when they are 14 days old.

* Internal improvements to the KP API and Widevine license server. Settings for each can ammended via a VUDRM Token.

* Internal improvements to the Config API. Failure to connect to the Database will now respond with a 5xx status code.

## January 2021

* Internal improvements to the ElasticSearch monitoring.

* Improvements to the Widevine license server - service has been re-written in Go.

* Various improvements to VUDRM Admin, including the ability to create a video for a client and JSON logging.

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
