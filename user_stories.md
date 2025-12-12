# User Stories

Title:
As an admin, I want to manage doctors and track appointment statistics, so that I can maintain the platform securely and efficiently.

Acceptance Criteria:

1.Admin must be able to log in using a valid username and password.

2.Admin must be able to log out securely from the portal.

3.Admin can add a new doctor by entering their required details.

4.Admin can delete a doctor’s profile from the portal.

5.Admin can run a stored procedure in MySQL CLI to view the number of appointments per month.

Priority: High
Story Points:
Notes:

Stored procedure returns monthly appointment count.

Ensure secure access control and proper session handling.






Title:
As a patient, I want to view a list of doctors without logging in, so that I can explore options before registering.

Acceptance Criteria:

1.Patient can access the doctor list without authentication.

2.Doctor list displays name, specialization, and availability.

3.Page loads without requiring sign-up or login.

Priority: Medium
Story Points:
Notes:

Ideal for new users evaluating the platform.






Title:
As a doctor, I want to log into the portal, so that I can manage my appointments.

Acceptance Criteria:

1.Doctor can log in using valid credentials.

2.Incorrect credentials display an error message.

3.Successful login redirects to the doctor dashboard.

Priority: High
Story Points:
Notes:

Secure authentication is required.
