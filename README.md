
## FrontEnd Implementation(Using React):
import React, { useState } from 'react';
import axios from 'axios';

function EnrollmentForm() {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    age: '',
    batch: '',
  });

  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setFormData({ ...formData, [name]: value });
  };

  const handleEnroll = async () => {
    try {
      
      const response = await axios.post('/api/enroll', formData);
      console.log(response.data);

      const paymentResponse = await axios.post('/api/payment', { enrollmentID: response.data.userID });
      console.log(paymentResponse.data);
    } catch (error) {
      console.error('Error:', error.response ? error.response.data : error.message);
    }
  };

  return (
    <div>
      <h2>Yoga Class Enrollment Form</h2>
      <form>
        <label>First Name:</label>
        <input type="text" name="firstName" onChange={handleInputChange} />

        <label>Last Name:</label>
        <input type="text" name="lastName" onChange={handleInputChange} />

        <label>Age:</label>
        <input type="number" name="age" onChange={handleInputChange} />

        <label>Batch:</label>
        <select name="batch" onChange={handleInputChange}>
          <option value="">Select Batch</option>
          <option value="6-7AM">6-7AM</option>
          <option value="7-8AM">7-8AM</option>
          <option value="8-9AM">8-9AM</option>
          <option value="5-6PM">5-6PM</option>
        </select>

        <button type="button" onClick={handleEnroll}>
          Enroll and Pay
        </button>
      </form>
    </div>
  );
}

export default EnrollmentForm;
## Backend Implementation:
const express = require('express');
const bodyParser = require('body-parser');
const app = express();
const PORT = 3000;

app.use(bodyParser.json());

const users = [];
const enrollments = [];
const payments = [];

app.post('/api/enroll', (req, res) => {
  
  const { firstName, lastName, age, batch } = req.body;
  if (!firstName || !lastName || !age || !batch) {
    return res.status(400).json({ error: 'Incomplete data provided.' });
  }

  
  if (age < 18 || age > 65) {
    return res.status(400).json({ error: 'Age must be between 18 and 65.' });
  }


  const user = { firstName, lastName, age };
  users.push(user);

  const enrollment = { userID: users.length, enrollDate: new Date(), batch };
  enrollments.push(enrollment);


  res.json({ success: true, message: 'Enrollment successful.' });
});

app.post('/api/payment', (req, res) => {
  const { enrollmentID } = req.body;

  const paymentResult = CompletePayment(enrollmentID);


  const payment = { enrollmentID, paymentDate: new Date(), amount: 500 };
  payments.push(payment);

 
  if (paymentResult === 'success') {
    res.json({ success: true, message: 'Payment successful.' });
  } else {
    res.status(400).json({ error: 'Payment failed.' });
  }
});

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});

## ER Diagram:
+-----------------+           +------------------+           +------------------+
|      User       |           |    Enrollment    |           |     Payment      |
+-----------------+           +------------------+           +------------------+
| UserID (PK)     |1        * | EnrollmentID (PK)|           | PaymentID (PK)   |
| FirstName       |-----------| UserID (FK)      |-------1   | EnrollmentID (FK)|
| LastName        |           | EnrollDate       |-----------| PaymentDate      |
| Age             |           | Batch            |           | Amount           |
+-----------------+           +------------------+           +------------------+

## Database Design:
1. User Table
- Store basic user information.
Column	   Data Type	  Constraints
UserID	   INT	        Primary Key, Auto-increment
FirstName	 VARCHAR(50)	Not Null
LastName	 VARCHAR(50)	Not Null
Age	       INT	        Check (18 <= Age <= 65)
2. Enrollment Table
Records each participant's enrollment details.
Column	      Data Type	    Constraints
EnrollmentID	INT	          Primary Key, Auto-increment
UserID	      INT         	Foreign Key (User.UserID)
EnrollDate	  DATE	        Not Null
Batch	        VARCHAR(20)	  Not Null
3. Payment Table
Records payment information for each participant.
Column	      Data Type	      Constraints
PaymentID	    INT           	Primary Key, Auto-increment
EnrollmentID	INT           	Foreign Key (Enrollment.EnrollmentID)
PaymentDate 	DATE          	Not Null
Amount	      DECIMAL(8,2)	  Check (Amount = 500.00)

## SQL Queries:
1. To Retrieve Participants in a Batch for a Given Month:
SELECT
    U.FirstName,
    U.LastName,
    E.EnrollDate,
    E.Batch
FROM
    User U
JOIN
    Enrollment E ON U.UserID = E.UserID
WHERE
    MONTH(E.EnrollDate) = @SelectedMonth
    AND E.Batch = @SelectedBatch;

2. To Insert a New Enrollment:
INSERT INTO Enrollment (UserID, EnrollDate, Batch)
VALUES (@UserID, @EnrollDate, @SelectedBatch);

3. To Record a Payment:
INSERT INTO Payment (EnrollmentID, PaymentDate, Amount)
VALUES (@EnrollmentID, @PaymentDate, 500.00);

4. To Switch Batch for the Next Month:
UPDATE
    Enrollment
SET
    Batch = @NewSelectedBatch
WHERE
    UserID = @UserID
    AND MONTH(EnrollDate) = @CurrentMonth;
