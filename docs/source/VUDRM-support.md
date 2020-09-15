# SUPPORT PROCESS

## How to contact us 

For all non-critical support request please email support@vualto.com. If you are a registered user you can contact support via the [help centre](https://vualto.zendesk.com). 

For all critical issues please contact support via one of the numbers below.

- +44 (0)800 0314391 
- +44 (0)1752 916051
- (800) 857 1808 (toll-free US number)

When contacting our support about a DRM issue please include as much relevant info as possible, this should include but not be limited to the [x-request-id](#x-request-id) for failing requests, the VUDRM token used, the VUDRM service, the DRM type/s, relevant content URL/s, response codes of the failed request, type of device/s the issue is occurring on, and browser type. If you can not get any of this information please state in you initial message that you are unable to provide this otherwise our first response will be asking for this information. 

### X-Request-ID

All requests made to and from VUDRM services will have a header called `x-request-id`. This header allows us to track a request through our backend; if you are getting unexpected errors from our services giving us the value of this header will allow us to efficiently track any errors that occurred while processing your request. 

### Vualto-Transaction-ID

On some requests you may find a header called `vualto-transaction-id`; this header was originally used for the same purpose as `x-request-id` header. In order to be more in line with industry standards, we are currently in the process of removing the `vualto-transaction-id` and replacing it with `x-request-id`. Please ensure that you use `x-request-id` contact support when contacting support and in any internal logging you perform.
