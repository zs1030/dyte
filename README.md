# dyte
#database schema for problem statement using postgresql
CREATE TABLE slots (
  id SERIAL PRIMARY KEY,
  start_time TIME NOT NULL,
  end_time TIME NOT NULL
);

CREATE TABLE faculties (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  department TEXT NOT NULL
);

CREATE TABLE courses (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  slot_id INTEGER REFERENCES slots(id),
  faculty_id INTEGER REFERENCES faculties(id)
);

CREATE TABLE students (
  id SERIAL PRIMARY KEY,
  registration TEXT UNIQUE NOT NULL,
  password TEXT NOT NULL,
  name TEXT NOT NULL
);

CREATE TABLE registrations (
  id SERIAL PRIMARY KEY,
  course_id INTEGER REFERENCES courses(id),
  student_id INTEGER REFERENCES students(id)
);
#let's implement the API routes and authentication endpoints using Node.js and Express.

Here is the code to set up our server and connect to the database.

const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const { Pool } = require('pg');

const app = express();
const port = process.env.PORT || 5000;

const pool = new Pool({
  user: 'postgres',
  host: 'localhost',
  database: 'ffcs',
  password: 'password',
  port: 5432,
});

pool.connect((err, client, done) => {
  if (err) {
    console.log('Error connecting to database', err);
  } else {
    console.log('Connected to database');
  }
});

app.use(cors());
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});

We will create the following API routes for the FFCS system:

/api/register - Register a new student or administrator.
/api/login - Log in as a student or administrator and receive a JWT.
/api/courses - View available courses for a student.
/api/register-courses - Register courses for a student.
/api/registered-courses - View registered courses for a student.
We will create a users table in the database to store the registration details of students and administrators.
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  registration_number VARCHAR(20) UNIQUE NOT NULL,
  password VARCHAR(60) NOT NULL,
  is_admin BOOLEAN NOT NULL DEFAULT false
);

Here is the code to register a new user.
app.post('/api/register', async (req, res) => {
  try {
    const { registrationNumber, password, isAdmin } = req.body;

    const hashedPassword = await bcrypt.hash(password, 10);

    const result = await pool.query(
      'INSERT INTO users (registration_number, password, is_admin) VALUES ($1, $2, $3) RETURNING id',
      [registrationNumber, hashedPassword, isAdmin]
    );

    const userId = result.rows[0].id;

    const token = jwt.sign({ userId, isAdmin }, 'secret-key');

    res.status(200).json({ token });
  } catch (err) {
    console.error(err);
    res.status(500).json({ message: 'Internal server error' });
  }
});
Here is the code to log in as a user and receive a JWT.
app.post('/api/login', async (req, res) => {
  try {
    const { registrationNumber, password } = req.body;

    const result = await pool.query('SELECT * FROM users WHERE registration_number = $1', [registrationNumber]);

    if (result.rows.length === 0) {
      res.status(401).json({ message: 'Invalid registration number or password' });
      return;
    }

    const user = result.rows[0];
    const isPasswordValid = await bcrypt.compare(password, user.password);

    if (!isPasswordValid) {
      res.status(401).json({ message: 'Invalid registration number or password' });
      return;
    }

    const token = jwt.sign({ userId: user.id, isAdmin: user.is_admin },

