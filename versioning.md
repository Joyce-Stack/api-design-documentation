Introduction 
Versioning is a controversial issue in the API community. Many of the approaches you will see in popular APIs such as Twitter or Google are now seen by many API puritans as just plain wrong. There are pros and cons to each approach. Some are complicated to implement; they are also complicated for your users. We have to pick an approach that is realistic for the project, that our target audience will be able to use and will support changes to the business requirements. 
Some criteria for picking a versioning strategy are Evolvability & Permalinking

Evolvability & Permalinking
One of goals of Data Spine (and therefore Funding Hub) is to be able to support making breaking changes on the server without the need to update all clients at the same time.  
The entities in each of the hubs will mint URIs that will be used across the organisation as a means of identifying an instance of an entity. These URIs are meant to be more like permalinks and it should never change. We should be able to lookup the URI at anytime with different versions and get the representation of that resource as outlined in that version, or request different representations (eg JSON-LD, RDF, Turtle etc). Maintaining URIs that do not change or break supports the ability to link together parts of the research graph. 

Non Breaking Changes 
Changes that are considered safe and do not require a version bump:-
Adding new API resources
Adding new optional request parameters to existing API methods
Adding new properties to existing API responses
Changing the order of properties in existing API responses
Changing the length or format of opaque strings eg integers to UUIDs 
(this might be controversial) 

Breaking Changes 
Removing properties 
Removing optional request parameters 
Legal reasons
If for any reasons that we are in violation of any legal policies and the companies reputation is at risk then a breaking change maybe necessary. We would not expect this to happen but if it did it would be sponsored by senior executive management.  
What if we need to make a backward breaking change?
When making backward-incompatible changes, offer both the old and an updated version for a time, before deprecating the old version. (TODO: a migration strategy) 
What if a client breaks with a 'backwards-compatible' change? 
While the Hub teams should endeavour to not introduce a breaking change, sometimes mistakes happen. If the Hub team does introduce a breaking change then they have an option to rollback or roll forward; whichever of these is most likely to fix the issue quicker. 
Hub teams can't be expected to wait for all clients to run all their test suites when an breaking change is required. Hub APIs should be intentionally designed so that improvements can be added and they can roll forward with features. 
In the unfortunate event that a change breaks then the policy should be not to rollback. One ill behaved client cannot block the progress of any part of the API development cycle. We recommend that clients program defensively 
Considerations
Ingesting different versioning details from different sources
Hub versioning 
per hub or 
per API within the Hub 
Schema changes 

Approaches
Custom Header in timestamp format
Each Hub would indicate their version in a custom header such as 'Hub-Version' but as a timestamp, not a numeric value such as 1.0, 3.0. This approach is about versioning the entire hub and not the individual endpoints within each hub.  
GET /awards/123456 HTTP/1.1						
Host: ds.elsevier.com
Hub-Version: 2014-05-04 

Pros: 
Straight forward enough for clients to understand 
Keeps versioning out of the URI therefore consistent URIs 
API communities agree that clients are less attached to timestamps than to version numbers - the jump from v1 to v2 feels like a big bump, 2015-04-29 to 2015-07-06 doesn't  
No need to version media types so everything could return application/ld+json 
Cons:
Breaks cachability as if the Vary header is not set on each response to indicate that the response is caceable - https://www.mnot.net/blog/2012/07/11/header_versioning and http://www.informit.com/articles/article.aspx?p=1566460
Versions the entire Hub API suite - a version bump means that all APIs in the Hub get bumped at the same time. The whole team have to work on bumping versions of all endpoints in the Hubs so doesn't allow for independent evolvability at an API level
Clients in the wild doing this:
GoGardless, Stripe and Azure 
Content Negotiation for the entire Hub 
Each hub defines their own media types that would be used in the Accept header to request a specific resource in a specific format. A numeric value (eg v1)  or a timestamp could be used. For the purpose of this I will use timestamps 
GET /awards/123456 HTTP/1.1						
Host: ds.elsevier.com
Accept: application/vnd.awards[.version-timestamp][+json] 
 
// Get the current version  
 
GET /awards/123456 HTTP/1.1						
Host: ds.elsevier.com
Accept: application/vnd.awards+json

 
// Get the version 2015-12-23 

GET /awards/123456 HTTP/1.1						
Host: ds.elsevier.com
Accept: application/vnd.awards.2015-12-23+json
 
// Get future versions preview 
GET /awards/123456 HTTP/1.1						
Host: ds.elsevier.com
Accept: application/vnd.awards.2020-12-23-preview+json
 
// Alternatives options for the Accept Header 
Accept: application/vnd.awards+json; version=2015-12-23 

Pros:
Gets around the caching problem outlined in previous approach - TODO show how 
HATEOAS friendly and semantically more correct 
Keeps URIs the same as no versioning in the URIs
You can have support multiple representations using one URI
Cons:
possible complicated routing logic (API gateway) 
not as obvious for some clients as approaches like versioning in the header or URI
a big bang approach to versioning - only a small percentage of the API may have changed so changing the entire API versions can cause worry in stakeholders. Would require diligent communications of changelogs and even then something may be missed and break a client 
can be mitigated by offering previews of upcoming releases and a careful communications strategy 
versions the entire Hub API suite - a version bump means that all APIs in the Hub get bumped at the same time. The whole team have to work on bumping versions of all endpoints in the Hubs so doesn't allow for independent evolvability at an API level
Clients in the wild doing this: 
GitHub 
Content Negotiation for each of the resources
Each hub will host several separate APIs. Each of these APIs will use content negotiation to determine the appropriate versions. to serve.  This is similar to the above approach but not at a hub level but at an API level. 

Pros:
HATEOAS friendly and semantically more correct 
Cache-friendly 
Keeps URIs the same as no versioning in the URIs 
You can have support multiple representations using one URI
Changes are localised to the API as opposed to the entire Hub. 
This means we could focus communication efforts on just those using that particular API as opposed to everybody in the organisation using Hubs. 
We have more chance of urging clients to upgrade their APIs 
The entire team does not have to be included in the versioning process, just those that are versioning the API in question. 
Cons:
Often seen as the most complex solution 
A bit more of a learning curve for clients but generally received ok once they get their head around it 
For a period of time both versions will need to exist side by side 
from the API producers point of view this is not impossible but it is difficult 
Clients using this in the wild:
GitHub
Version Details
All hubs should have a /version endpoint which shows all details about the data used (eg data source versions, schema versions, ontologies versions etc )
All endpoints should have a /status endpoint which shows if the API is operational or not 