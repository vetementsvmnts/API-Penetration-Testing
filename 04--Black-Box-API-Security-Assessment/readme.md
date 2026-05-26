Project Overview
This project demonstrates the discovery and exploitation of a Broken Object Level Authorization (BOLA)—also known as Insecure Direct Object Reference (IDOR)—vulnerability within the crAPI (Completely Ridiculous API) ecosystem. By analyzing API requests and gathering target information via community features, an authenticated user can access private location data and personal details of other registered users.

Technical Description of the Attack
The attack was executed in three primary phases: Information Gathering, Traffic Analysis, and Exploitation.

1. Information Gathering
Target Identification: Navigated to the crAPI Community Forum (/forum) to find potential target users.

Target Selection: Identified a user named Pogba who had active posts on the platform.

2. Traffic Analysis
Interception: Used Burp Suite Community Edition to capture legitimate API traffic on the dashboard.

Endpoint Discovery: Located the vehicle details page, which triggers a GET request to retrieve the current user's vehicle location via the following API endpoint structure:
GET /identity/api/v2/vehicle/<carId>/location

Authentication: The request utilizes a standard Bearer Token (Authorization: Bearer <JWT>) for user authentication. However, the backend fails to validate if the authenticated user actually owns the requested <carId>.

3. Exploitation (BOLA / IDOR)
Request Manipulation: Sent the legitimate location request to Burp Repeater.

IDOR Execution: Swapped out the original user's carId with the target user's (Pogba) carId (retrieved from the API/forum context: cd515c12-0fc1-48ae-8b61-9230b70a845b).

Data Leakage: The server responded with a 200 OK, exposing Pogba's sensitive information:

GPS Coordinates: Latitude (31.284788), Longitude (-92.471176)

Full Name: Pogba

Email Address: pogbaU06@example.com

Exploit Proof of Concept (PoC)
Vulnerable Endpoint
HTTP
GET /identity/api/v2/vehicle/{car_id}/location HTTP/1.1
Host: localhost:8888
Authorization: Bearer [Your_JWT_Token]
Response (Data Leak)
JSON
{
  "carId": "cd515c12-0fc1-48ae-8b61-9230b70a845b",
  "vehicleLocation": {
    "id": 2,
    "latitude": "31.284788",
    "longitude": "-92.471176"
  },
  "fullName": "Pogba",
  "email": "pogbaU06@example.com"
}
Remediation & Mitigation
To secure this endpoint against BOLA/IDOR attacks, implement the following controls:

Object-Level Access Control: Implement an authorization check on the backend to verify that the carId specified in the URL path belongs explicitly to the user ID extracted from the validated JWT token.

Use Non-Sequential/Non-Leaked Identifiers: While crAPI uses UUIDs, ensure that asset IDs are not leaked unnecessarily through public endpoints like the community forum.

Principle of Least Privilege: Ensure the API endpoint only returns the data necessary for the request. If only the location is needed, do not include the owner's full name and email address in the JSON payload response.

Environment & Tools Used
Target Application: crAPI (Completely Ridiculous API) hosted on localhost:8888

Interception Proxy: Burp Suite Community Edition v2026.4.3

Mail Server: MailHog (for monitoring local registration/token mail flows)
