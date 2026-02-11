# API Testing Essentials - A Complete Guide
*By Sadath EM - QA Test Lead*

## Introduction

API testing has become increasingly important in modern software development. Throughout my career testing microservices architectures, payment systems, and healthcare integrations, I've learned that robust API testing is crucial for ensuring system reliability. This guide covers everything from basic API concepts to advanced testing strategies using Postman and REST Assured.

## 1. Why API Testing?

### Advantages over UI Testing:
- ⚡ **Faster**: No UI rendering delays
- 🎯 **More Stable**: No UI element changes
- 🔍 **Better Coverage**: Test business logic directly
- 🔧 **Early Testing**: Test before UI is ready
- 💰 **Cost-Effective**: Easier to maintain
- 🚀 **CI/CD Friendly**: Easy to integrate

### Real-World Use Cases from My Experience:

**Banking (Deutsche Bank)**
- Payment initiation APIs
- SWIFT message validation
- Transaction status verification
- Credit bureau integrations

**Healthcare (Tieto EVRY)**
- Patient order management APIs
- HL7 message processing
- External system integrations

**Telecom (British Telecom)**
- Number portability APIs
- Service activation endpoints
- Billing system integrations

## 2. API Basics

### HTTP Methods

```
GET     - Retrieve data
POST    - Create new resource
PUT     - Update entire resource
PATCH   - Partial update
DELETE  - Remove resource
HEAD    - Get headers only
OPTIONS - Get supported methods
```

### HTTP Status Codes

```
Success Codes:
200 OK                  - Request successful
201 Created             - Resource created
204 No Content          - Successful, no response body

Client Error Codes:
400 Bad Request         - Invalid request
401 Unauthorized        - Authentication required
403 Forbidden           - No permission
404 Not Found           - Resource doesn't exist
409 Conflict            - Conflicting request
422 Unprocessable       - Validation failed

Server Error Codes:
500 Internal Server     - Server error
502 Bad Gateway         - Invalid response from upstream
503 Service Unavailable - Server temporarily down
504 Gateway Timeout     - Upstream server timeout
```

### Common Headers

```
Request Headers:
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
User-Agent: PostmanRuntime/7.29.0

Response Headers:
Content-Type: application/json
Content-Length: 1234
Cache-Control: no-cache
Set-Cookie: sessionId=abc123
```

### Request/Response Format

**JSON Request**
```json
{
  "username": "test_user",
  "email": "test@example.com",
  "password": "Test@123"
}
```

**JSON Response**
```json
{
  "status": "success",
  "message": "User created successfully",
  "data": {
    "userId": 101,
    "username": "test_user",
    "email": "test@example.com",
    "createdAt": "2024-02-11T10:30:00Z"
  }
}
```

## 3. Postman - Getting Started

### Installation
1. Download from [postman.com](https://www.postman.com/downloads/)
2. Install and create account
3. Create workspace and collection

### Basic Request

**Step 1: Create Collection**
- Click "New Collection"
- Name it "API Tests"

**Step 2: Add Request**
- Click "Add Request"
- Name it "Get All Users"
- Select GET method
- Enter URL: `https://api.example.com/users`
- Click "Send"

### Setting Up Environment

**Step 1: Create Environment**
- Click "Environments" tab
- Click "Create Environment"
- Name it "QA Environment"

**Step 2: Add Variables**
```
Variable Name    Initial Value              Current Value
baseUrl          https://qa.api.example.com https://qa.api.example.com
authToken        your_auth_token_here       your_auth_token_here
userId           101                        101
```

**Step 3: Use Variables in Requests**
```
URL: {{baseUrl}}/users/{{userId}}
Headers:
  Authorization: Bearer {{authToken}}
```

### Authentication in Postman

**Bearer Token**
```
Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Basic Auth**
```
Authorization Tab:
  Type: Basic Auth
  Username: admin
  Password: admin123
```

**API Key**
```
Headers:
  X-API-Key: your_api_key_here
```

### Writing Tests in Postman

**Test Script Tab:**

```javascript
// 1. Status code validation
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// 2. Response time validation
pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// 3. Response body validation
pm.test("Response has userId", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('userId');
});

// 4. Specific value validation
pm.test("User status is active", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql('active');
});

// 5. Array validation
pm.test("Response contains users array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.users).to.be.an('array');
    pm.expect(jsonData.users).to.have.lengthOf.at.least(1);
});

// 6. Header validation
pm.test("Content-Type is application/json", function () {
    pm.response.to.have.header("Content-Type", "application/json");
});

// 7. Extract and save data
pm.test("Extract userId", function () {
    var jsonData = pm.response.json();
    pm.environment.set("userId", jsonData.userId);
});

// 8. Validate JSON schema
const schema = {
    "type": "object",
    "properties": {
        "userId": {"type": "number"},
        "username": {"type": "string"},
        "email": {"type": "string"}
    },
    "required": ["userId", "username", "email"]
};

pm.test("Schema is valid", function() {
    pm.response.to.have.jsonSchema(schema);
});
```

### Pre-request Scripts

```javascript
// Generate random data
pm.environment.set("randomEmail", 
    "test" + Math.floor(Math.random() * 10000) + "@example.com"
);

// Set timestamp
pm.environment.set("timestamp", new Date().toISOString());

// Generate dynamic values
const uuid = require('uuid');
pm.environment.set("uniqueId", uuid.v4());

// Set authentication token
const moment = require('moment');
pm.environment.set("expiryDate", moment().add(1, 'days').format());
```

## 4. Real-World API Test Scenarios

### Test Case 1: User Registration

**Request:**
```
POST {{baseUrl}}/api/users/register

Headers:
Content-Type: application/json

Body:
{
  "username": "newuser_{{$randomInt}}",
  "email": "newuser_{{$randomInt}}@test.com",
  "password": "Test@123",
  "firstName": "Test",
  "lastName": "User"
}
```

**Tests:**
```javascript
// Positive scenario
pm.test("User registration successful - Status 201", function () {
    pm.response.to.have.status(201);
});

pm.test("Response contains userId", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.data.userId).to.exist;
    pm.environment.set("newUserId", jsonData.data.userId);
});

pm.test("Username matches request", function () {
    var jsonData = pm.response.json();
    var requestData = JSON.parse(pm.request.body.raw);
    pm.expect(jsonData.data.username).to.eql(requestData.username);
});

// Negative scenario - Duplicate email
pm.test("Duplicate email returns 409 Conflict", function () {
    pm.response.to.have.status(409);
    var jsonData = pm.response.json();
    pm.expect(jsonData.message).to.include("already exists");
});
```

### Test Case 2: Login API

**Request:**
```
POST {{baseUrl}}/api/auth/login

Body:
{
  "username": "test_user",
  "password": "Test@123"
}
```

**Tests:**
```javascript
pm.test("Login successful", function () {
    pm.response.to.have.status(200);
});

pm.test("Response contains access token", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.accessToken).to.exist;
    pm.expect(jsonData.refreshToken).to.exist;
    
    // Save token for subsequent requests
    pm.environment.set("authToken", jsonData.accessToken);
    pm.environment.set("refreshToken", jsonData.refreshToken);
});

pm.test("Token expiry is set", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.expiresIn).to.be.above(0);
});

// Invalid credentials test
pm.test("Invalid credentials return 401", function () {
    pm.response.to.have.status(401);
    var jsonData = pm.response.json();
    pm.expect(jsonData.message).to.eql("Invalid credentials");
});
```

### Test Case 3: Payment Processing (Banking Domain)

**Request:**
```
POST {{baseUrl}}/api/payments/initiate

Headers:
Authorization: Bearer {{authToken}}
Content-Type: application/json

Body:
{
  "accountNumber": "1234567890",
  "amount": 1000.50,
  "currency": "USD",
  "beneficiaryAccount": "9876543210",
  "beneficiaryName": "John Doe",
  "purpose": "Payment",
  "transactionType": "RTGS"
}
```

**Tests:**
```javascript
pm.test("Payment initiated successfully", function () {
    pm.response.to.have.status(200);
});

pm.test("Transaction ID generated", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.transactionId).to.exist;
    pm.expect(jsonData.transactionId).to.match(/^TXN[0-9]{10}$/);
    pm.environment.set("transactionId", jsonData.transactionId);
});

pm.test("Payment status is pending", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql("PENDING");
});

pm.test("Amount matches request", function () {
    var jsonData = pm.response.json();
    var requestData = JSON.parse(pm.request.body.raw);
    pm.expect(jsonData.amount).to.eql(requestData.amount);
});

// Insufficient balance test
pm.test("Insufficient balance returns 422", function () {
    pm.response.to.have.status(422);
    var jsonData = pm.response.json();
    pm.expect(jsonData.error).to.include("Insufficient balance");
});

// Validate SWIFT message (from Deutsche Bank experience)
pm.test("SWIFT message format is valid", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.swiftMessage).to.exist;
    pm.expect(jsonData.swiftMessage).to.include("MT103");
});
```

### Test Case 4: Healthcare Order Management

**Request:**
```
POST {{baseUrl}}/api/orders/create

Headers:
Authorization: Bearer {{authToken}}

Body:
{
  "patientId": "PAT123456",
  "orderType": "LAB_TEST",
  "items": [
    {
      "testCode": "CBC",
      "testName": "Complete Blood Count",
      "priority": "ROUTINE"
    }
  ],
  "physicianId": "DOC789",
  "notes": "Fasting required"
}
```

**Tests:**
```javascript
pm.test("Order created successfully", function () {
    pm.response.to.have.status(201);
});

pm.test("Order ID generated", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.orderId).to.exist;
    pm.environment.set("orderId", jsonData.orderId);
});

pm.test("Patient data privacy maintained", function () {
    var jsonData = pm.response.json();
    // Ensure sensitive patient data is not in response
    pm.expect(jsonData).to.not.have.property('ssn');
    pm.expect(jsonData).to.not.have.property('dateOfBirth');
});

pm.test("HL7 message generated", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.hl7Message).to.exist;
    pm.expect(jsonData.hl7Message).to.include("ORM^O01");
});
```

### Test Case 5: Telecom Service Activation

**Request:**
```
POST {{baseUrl}}/api/services/activate

Body:
{
  "customerId": "CUST123456",
  "serviceType": "BROADBAND",
  "plan": "FIBER_100MBPS",
  "installationAddress": {
    "street": "123 Main St",
    "city": "Bangalore",
    "zipCode": "560001"
  }
}
```

**Tests:**
```javascript
pm.test("Service activation initiated", function () {
    pm.response.to.have.status(200);
});

pm.test("Service order number generated", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.orderNumber).to.exist;
    pm.environment.set("serviceOrderNumber", jsonData.orderNumber);
});

pm.test("Installation date scheduled", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.installationDate).to.exist;
    
    // Verify installation date is in future
    var installDate = new Date(jsonData.installationDate);
    var today = new Date();
    pm.expect(installDate).to.be.above(today);
});
```

## 5. Advanced Postman Features

### Collection Runner

**Setup:**
1. Click "Runner" button
2. Select collection
3. Select environment
4. Set iterations
5. Upload data file (CSV/JSON)
6. Run collection

**Data File (CSV):**
```csv
username,password,expectedStatus
valid_user,valid_pass,200
invalid_user,invalid_pass,401
empty_user,,400
```

**Using Data in Request:**
```javascript
// In request body
{
  "username": "{{username}}",
  "password": "{{password}}"
}

// In tests
pm.test("Status matches expected", function () {
    pm.response.to.have.status(parseInt(data.expectedStatus));
});
```

### Newman - Command Line Runner

```bash
# Install Newman
npm install -g newman

# Run collection
newman run collection.json -e environment.json

# With reporters
newman run collection.json -e environment.json -r html,cli

# With iterations
newman run collection.json -e environment.json -n 10

# With data file
newman run collection.json -e environment.json -d data.csv
```

### Monitors

Setup automated API monitoring:
1. Click "Monitors" tab
2. Create new monitor
3. Select collection
4. Set schedule (hourly, daily, weekly)
5. Configure notifications

## 6. REST Assured - Java Framework

### Setup

**pom.xml**
```xml
<dependencies>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>5.3.2</version>
        <scope>test</scope>
    </dependency>
    
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.8.0</version>
    </dependency>
    
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.15.2</version>
    </dependency>
</dependencies>
```

### Basic GET Request

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;
import org.testng.annotations.Test;

public class APITest {
    
    @Test
    public void testGetUser() {
        given()
            .baseUri("https://api.example.com")
            .header("Content-Type", "application/json")
        .when()
            .get("/users/101")
        .then()
            .statusCode(200)
            .body("userId", equalTo(101))
            .body("username", notNullValue())
            .body("email", containsString("@"));
    }
}
```

### POST Request with Body

```java
@Test
public void testCreateUser() {
    String requestBody = "{\n" +
        "  \"username\": \"test_user\",\n" +
        "  \"email\": \"test@example.com\",\n" +
        "  \"password\": \"Test@123\"\n" +
        "}";
    
    given()
        .baseUri("https://api.example.com")
        .header("Content-Type", "application/json")
        .body(requestBody)
    .when()
        .post("/users")
    .then()
        .statusCode(201)
        .body("status", equalTo("success"))
        .body("data.userId", notNullValue());
}
```

### Using POJOs

**User.java**
```java
public class User {
    private String username;
    private String email;
    private String password;
    
    // Constructors, getters, setters
    public User(String username, String email, String password) {
        this.username = username;
        this.email = email;
        this.password = password;
    }
    
    // Getters and setters...
}
```

**Test with POJO:**
```java
@Test
public void testCreateUserWithPOJO() {
    User user = new User("test_user", "test@example.com", "Test@123");
    
    given()
        .baseUri("https://api.example.com")
        .contentType("application/json")
        .body(user)
    .when()
        .post("/users")
    .then()
        .statusCode(201)
        .body("data.username", equalTo(user.getUsername()));
}
```

### Response Extraction

```java
@Test
public void testExtractResponse() {
    Response response = given()
        .baseUri("https://api.example.com")
        .contentType("application/json")
    .when()
        .get("/users/101")
    .then()
        .statusCode(200)
        .extract().response();
    
    // Extract specific values
    int userId = response.path("userId");
    String username = response.path("username");
    
    // Extract as POJO
    User user = response.as(User.class);
    
    System.out.println("User ID: " + userId);
    System.out.println("Username: " + username);
}
```

### Path Parameters

```java
@Test
public void testPathParameters() {
    given()
        .baseUri("https://api.example.com")
        .pathParam("userId", 101)
    .when()
        .get("/users/{userId}")
    .then()
        .statusCode(200);
}
```

### Query Parameters

```java
@Test
public void testQueryParameters() {
    given()
        .baseUri("https://api.example.com")
        .queryParam("status", "active")
        .queryParam("limit", 10)
    .when()
        .get("/users")
    .then()
        .statusCode(200)
        .body("users.size()", lessThanOrEqualTo(10));
}
```

## 7. API Testing Best Practices

### Test Coverage

**What to Test:**
1. ✅ Status codes (200, 201, 400, 401, 404, 500)
2. ✅ Response time
3. ✅ Response headers
4. ✅ Response body structure
5. ✅ Data validation
6. ✅ Error messages
7. ✅ Authentication/Authorization
8. ✅ Input validation (positive & negative)
9. ✅ Boundary conditions
10. ✅ Data integrity

### Test Organization

```
API Tests/
├── Authentication/
│   ├── Login
│   ├── Logout
│   └── Refresh Token
├── User Management/
│   ├── Create User
│   ├── Get User
│   ├── Update User
│   └── Delete User
├── Orders/
│   ├── Create Order
│   ├── Get Order
│   └── Cancel Order
└── Payments/
    ├── Initiate Payment
    ├── Get Payment Status
    └── Refund Payment
```

### Naming Conventions

```
✅ Good:
- GET_Users_Returns200_WhenUsersExist
- POST_User_Returns201_WhenValidDataProvided
- POST_Login_Returns401_WhenInvalidCredentials

❌ Bad:
- Test1
- API_Test
- CheckUsers
```

### Data Management

```javascript
// Use environment variables
{{baseUrl}}, {{authToken}}, {{userId}}

// Generate dynamic data
{{$guid}}, {{$timestamp}}, {{$randomInt}}

// Use pre-request scripts for complex data
pm.environment.set("uniqueEmail", 
    "user_" + Date.now() + "@test.com"
);
```

## 8. Common Challenges and Solutions

### Challenge 1: Authentication Token Expiry

**Solution:**
```javascript
// Pre-request script to refresh token
const tokenExpiry = pm.environment.get("tokenExpiry");
const currentTime = new Date().getTime();

if (!tokenExpiry || currentTime > tokenExpiry) {
    // Get new token
    pm.sendRequest({
        url: pm.environment.get("baseUrl") + "/auth/refresh",
        method: 'POST',
        header: {
            'Content-Type': 'application/json',
        },
        body: {
            mode: 'raw',
            raw: JSON.stringify({
                refreshToken: pm.environment.get("refreshToken")
            })
        }
    }, function (err, response) {
        const newToken = response.json().accessToken;
        const expiresIn = response.json().expiresIn;
        pm.environment.set("authToken", newToken);
        pm.environment.set("tokenExpiry", currentTime + (expiresIn * 1000));
    });
}
```

### Challenge 2: Dependent API Calls

**Solution: Chain requests**
```javascript
// Test 1: Create user, save userId
pm.environment.set("userId", jsonData.userId);

// Test 2: Use saved userId
GET {{baseUrl}}/users/{{userId}}
```

### Challenge 3: Dynamic Data Validation

**Solution:**
```javascript
pm.test("Validate dynamic timestamp", function () {
    var jsonData = pm.response.json();
    var timestamp = new Date(jsonData.createdAt);
    var now = new Date();
    var diff = now - timestamp;
    
    // Timestamp should be within last 5 seconds
    pm.expect(diff).to.be.below(5000);
});
```

## 9. Performance Considerations

### Response Time Testing

```javascript
pm.test("Response time under 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// Log response time
console.log("Response time: " + pm.response.responseTime + "ms");
```

### Load Testing with Postman

Use Collection Runner:
- Set iterations: 100
- Set delay: 0ms
- Monitor response times

For serious load testing, use:
- Apache JMeter
- Gatling
- K6

## 10. Documentation

### Postman Documentation Features

1. Add description to collections
2. Add request examples
3. Add test documentation
4. Generate API documentation
5. Share with team

### Example Documentation

```markdown
# User Management API

## Create User
Creates a new user in the system.

**Endpoint:** POST /api/users

**Headers:**
- Content-Type: application/json
- Authorization: Bearer {token}

**Request Body:**
{
  "username": "string",
  "email": "string",
  "password": "string"
}

**Response:**
201 Created
{
  "status": "success",
  "data": {
    "userId": 101,
    "username": "test_user",
    "email": "test@example.com"
  }
}

**Error Responses:**
- 400 Bad Request: Invalid input
- 409 Conflict: User already exists
- 500 Internal Server Error: Server error
```

## Conclusion

API testing is crucial for ensuring the reliability and functionality of modern applications. Whether you're testing payment systems, healthcare integrations, or telecom services, the principles remain the same:

1. Test thoroughly - positive, negative, boundary cases
2. Validate comprehensively - status, headers, body, performance
3. Automate wisely - use Postman collections and Newman
4. Document properly - help your team understand the APIs
5. Integrate with CI/CD - run tests automatically

---

## Resources

Download my complete Postman collection with real-world examples from the Resources section.

---

*Questions about API testing? [Get in touch](../contact.html).*

**Tags**: #APITesting #Postman #RESTAssured #QA #Automation #Testing
