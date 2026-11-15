# CICD-1 Coding Test — Review, Improve and Reflect

**Student ID:** G00464215  
**Examination group:** A (Room Booking / JPQL) or B (OpenFeign): B (OpenFeign) <br>
**Original examination pull-request URL (`exam-work`):** https://github.com/RonsJose/cicd1-exam2_BookingService/pull/1 <br>
**New review repository URL:** https://github.com/RonsJose/CICD1_Review_G00464215

## 1. My original examination self-assessment

Review the code **as submitted in the original examination**. Do not change the original repository, branch or pull request. Check your own Group A or Group B examination paper and award marks only for evidence present in the original submission.

### Task 1 — /40
**My mark:** 30<br>
**Evidence from the original code (files, methods, working or missing requirements):**<br>
I've got the booking model with the validation annotations, the request and response DTOs. The DTO's have the required fields mentioned in the test file.<br>

However, what I've done wrong is with the validation on the name length, there is no @Size(min = 2, max = 60) to restrict the length of the name only @NotBlank to ensure it is not blank.<br>
@NotBlank<br>
private String name;<br>

I also don't use the @Valid parameter in the method when creating the bookings which means that it does not actually check to see if the rules are checked.<br>
@PostMapping<br>
@ResponseStatus(HttpStatus.CREATED)<br>
public Booking create(@RequestBody Request booking) {<br>
return  service.create(booking);<br>
}<br>

There is no validation on the DTO itself, only on the booking model. The only reason that the request fails when you put in information that violates the rules is because of the @Entity annotation on the booking model, and returns a 500 Internal server error, instead of 400 Bad Request.<br>
public record Request(String name, Long roomID, int count) {
}
### Task 2 — /40
**My mark:** 30 <br>
**Evidence from the original code:**<br>
I use OpenFeign to retrieve the selected room from the endpoint and reject the bookings where the attendees exceed the room capacity. But I did not catch the exception thrown when an incorrect id for room is put in.<br>
Room room = bookingClient.getRoom(request.roomID());<br>
if (request.count()> room.getCapacity())<br>
{<br>
throw new ResponseStatusException(HttpStatus.CONFLICT, "Room Capacity Exceeded");<br>
}<br>

I've also used HTTP response codes although only the one for successful creation was used right, for the attendees exceeding the room capacity I used conflict instead of invalid input.
@ResponseStatus(HttpStatus.CREATED)<br>
public Booking create(@RequestBody Request booking) {<br>
return  service.create(booking);<br>
}<br>
throw new ResponseStatusException(HttpStatus.CONFLICT, "Room Capacity Exceeded");<br>

The post body only includes the room id, name, and count of attendees.<br>
public record Request(String name, Long roomID, int count) {<br>
}
### Task 3 — /20
**My mark:** 20  <br>
**Evidence from the original code:**<br>
I check the local database for the id of the booking and if not found return a 404 http status code with the message "Incorrect booking id".<br>
Booking booking  = bookingRepository.findById(id).orElseThrow(()-> new ResponseStatusException(HttpStatus.NOT_FOUND,"Incorrect booking id"));<br>

It is returned using the response DTO.<br>
Responses response = new Responses();<br>
response.setId(booking.getId());<br>
response.setName(booking.getName());<br>
response.setRoomID(booking.getRoomID());<br>
response.setCount(booking.getCount());<br>
response.setStatus(booking.isStatus());<br>
return response;<br>

**Total self-assessed original examination mark: 80/100**

**Checkpoint:** Commit this completed self-assessment as the **first review commit in your new repository**, before you modify or add application code. The lecturer decides the official original examination mark.

## 2. Corrections and improvements in my new project

Start from a **fresh copy of the starter template for your own exam group**, in a **new GitHub repository**. You may consult or selectively reuse your own original examination code, but the new project must demonstrate that you understand and can correct it. Do not change your original examination submission.

Describe **two or three important problems** in your original code. For each, explain what was wrong, which class or method you changed in the new project, and why the corrected approach works.

**Problem and correction 1:**
The validation for the request dto was not setup, so I changed the request record to include the appropriate validation annotations, this way it actually looks at the dto for rules instead of giving internal server errors when it tries to save the information from the request to the database and instead gives 400 bad request code.<br>
The annotation thats needed in the controller to actually run the verification to make sure that the input followed the rules was also not there so I added the @Valid annotation to the create method in the booking controller.

**Problem and correction 2:**
Try catch if the room id of a booking does not exist, try the method of getting the id by OpenFeign call if that returns an error catch it and throw a new status exception.


**Problem and correction 3 (optional):**
I was returning a 409 a conflict, error code when the number of attendees exceeded the capacity of a room instead of the appropriate 400 bad request code. So I just switched it to 400.

## 3. API testing and evidence

Use Swagger UI or another HTTP client. For **at least four meaningful checks**, record the request/input, expected result, actual HTTP status/response, and pass/fail. Include unsuccessful cases; a running application alone is not enough.

**Test 1 — successful booking request:**
- Request / input:<br>
{<br>
  "name": "SuperMacs",<br>
  "roomID": 1,<br>
  "count": 15<br>
  }<br>
- Expected: 201
- Actual HTTP status / body: 201 
Pass / fail: Pass

**Test 2 — invalid input or excessive attendee count:**
- Request / input:<br>
  {<br>
  "name": "Dunnes",<br>
  "roomID": 1,<br>
  "count": 500<br>
  }<br>
- Expected: 400
- Actual HTTP status / body: 400
- Pass / fail: Pass

**Test 3 — missing booking or room:**
- Request / input:<br>
{<br>
  "name": "Tesco",<br>
  "roomID": 245,<br>
  "count": 5<br>
  }<br>
- Expected: 404
- Actual HTTP status / body: 404
- Pass / fail: Pass

**Test 4 — requirement specific to my group:**
- Request / input:<br>
{<br>
  "name": "Tesco",<br>
  "roomID": 2,<br>
  "count": 11<br>
  }<br>
- Expected: 400
- Actual HTTP status / body: 400
- Pass / fail: Pass

**Test 5 — requirement specific to my group:**
- Request / input: 3
- Expected: 404
- Actual HTTP status / body: 404
- Pass / fail: Pass

**Test 6 — requirement specific to my group:**
- Request / input: 1
- Expected: 200<br>
  {<br>
  "id": 1,<br>
  "name": "SuperMacs",<br>
  "count": 15,<br>
  "roomID": 1,<br>
  "status": true<br>
  }<br>
- Actual HTTP status / body: 200<br>
  {<br>
  "id": 1,<br>
  "name": "SuperMacs",<br>
  "count": 15,<br>
  "roomID": 1,<br>
  "status": true<br>
  }<br>
- Pass / fail: Pass

**Group A:** Demonstrate `GET /api/bookings/search?roomName=...` using an exact room name, bookings ordered by attendee count descending, and an empty list for no matches. Add checks if necessary to show all three behaviours.

**Group B:** Run the **provided** Room Service separately on port 8082 and the **new** Booking Service on port 8081. Demonstrate that booking creation obtains room details through OpenFeign and that booking retrieval uses the local Booking database. Do not modify the Room Service.

## 4. Reflection

**What did I understand during the examination but struggle to implement?**

**Which Spring Boot, JPA, validation, JPQL or OpenFeign concept became clearer after correcting my code? Give a specific example.**

**What will I do differently in my next coding test or API development task?**

## Review lab assessment — 20 marks

- Honest original self-assessment with code evidence: **4 marks**
- Explanation of mistakes and Spring Boot concepts: **6 marks**
- Corrections in the new application: **4 marks**
- API testing and demonstration of understanding: **4 marks**
- Reflection and future improvement: **2 marks**

**Total: 20 marks.** Simply submitting a working solution does not demonstrate the learning required by this review.

## Proposed mark recovery — only if confirmed for CICD-1

If the same approach as OOP is adopted, the review could recover **up to 30% of marks lost** in the original examination, scaled by the review-lab mark.

**Marks recovered = (100 − official original exam mark) × 0.30 × (review lab mark ÷ 20)**

Example: exam **40/100** and review **16/20** → **14.4 recovered marks**, revised exam **54.4/100**. This is **not an approved CICD-1 policy unless confirmed by the lecturer**. The original official exam mark—not your self-awarded grade—is used in the calculation.

## Submission

Commit and push the **new review repository** with the completed README, corrected Spring Boot code and API-test evidence. Submit its GitHub URL on Moodle. **Do not push any changes to the original examination repository or `exam-work` pull request.**