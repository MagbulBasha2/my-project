Here’s the **organized project structure** for your **Employee Management System** (MERN Stack), with **all filenames**, **proper folder arrangement**, and a **single `README.md`** file that clearly explains everything — from setup to usage.

---

## 📂 Project Structure

```
my-project/
│
├── backend/
│   ├── models/
│   │   └── Employee.js
│   ├── server.js
│   ├── package.json
│
├── frontend/
│   ├── src/
│   │   ├── App.js
│   │   ├── AddEmployee.js
│   │   ├── SearchEmployee.js
│   │   ├── UpdateEmployee.js
│   │   ├── DisplayAll.js
│   │   ├── DeleteEmployee.js
│   │   └── index.js
│   ├── package.json
│
└── README.md
```

---

## 📘 README.md

````markdown
# Employee Management System (MERN Stack)

This project is a full-stack **Employee Management System** built using the **MERN Stack (MongoDB, Express.js, React.js, Node.js)**.  
It allows users to perform CRUD (Create, Read, Update, Delete) operations on employee records.

---

## 🚀 Features

- Add a new employee  
- Search employee by ID  
- Update employee designation  
- View all employees  
- Delete employee by ID  

---

## 🧩 Technologies Used

### Backend
- Node.js  
- Express.js  
- MongoDB (via Mongoose)

### Frontend
- React.js  
- Axios  
- Bootstrap  

---

## ⚙️ Installation & Setup Guide

### 1️⃣ Clone the Repository
```bash
git clone https://github.com/yourusername/my-project.git
cd my-project
````

---

### 2️⃣ Setup Backend (Server)

```bash
cd backend
npm install
```

#### Start MongoDB (if not already running)

```bash
mongod
```

#### Run Server

```bash
node server.js
```

Server will run at: **[http://localhost:5000](http://localhost:5000)**

---

### 3️⃣ Setup Frontend (React App)

```bash
cd ../frontend
npm install
npm start
```

Frontend runs at: **[http://localhost:3000](http://localhost:3000)**

---

## 📦 Backend Code Overview

### `backend/models/Employee.js`

```js
const mongoose = require('mongoose');

const employeeSchema = new mongoose.Schema({
  EmployeeName: { type: String, required: true },
  EmployeeID: { type: String, required: true, unique: true },
  Designation: { type: String, required: true },
  Department: { type: String, required: true },
  JoiningDate: { type: Date, required: true }
});

const Employee = mongoose.model('Employee', employeeSchema);
module.exports = Employee;
```

---

### `backend/server.js`

```js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const Employee = require('./models/Employee');

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://127.0.0.1:27017/Employees')
  .then(() => console.log('MongoDB Database is Connected'))
  .catch(err => console.log('DB Not Connected ' + err));

app.post('/employees', async (req, res) => {
  try {
    const employee = new Employee(req.body);
    await employee.save();
    res.status(201).send({ message: 'Employee added successfully' });
  } catch (error) {
    res.status(400).send({ error: error.message });
  }
});

app.get('/employees/:EmployeeID', async (req, res) => {
  try {
    const emp = await Employee.findOne({ EmployeeID: req.params.EmployeeID });
    if (!emp) return res.status(404).send({ error: 'Employee not found' });
    res.send(emp);
  } catch (error) {
    res.status(500).send({ error: error.message });
  }
});

app.put('/employees/:EmployeeID', async (req, res) => {
  try {
    const updated = await Employee.findOneAndUpdate(
      { EmployeeID: req.params.EmployeeID },
      { Designation: req.body.Designation },
      { new: true }
    );
    if (!updated)
      return res.status(404).send({ error: 'Employee not found' });
    res.send({ message: 'Designation updated successfully', employee: updated });
  } catch (error) {
    res.status(500).send({ error: error.message });
  }
});

app.delete('/employees/:EmployeeID', async (req, res) => {
  try {
    const delEmp = await Employee.findOneAndDelete({ EmployeeID: req.params.EmployeeID });
    if (!delEmp)
      return res.status(404).send({ error: 'Employee not found' });
    res.send({ message: 'Employee deleted successfully' });
  } catch (error) {
    res.status(500).send({ error: error.message });
  }
});

app.get('/employees', async (req, res) => {
  try {
    const emp = await Employee.find();
    res.send(emp);
  } catch (error) {
    res.status(500).send({ error: error.message });
  }
});

const PORT = 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

---

## 💻 Frontend Code Overview

### `frontend/src/App.js`

```js
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
import AddEmployee from './AddEmployee';
import SearchEmployee from './SearchEmployee';
import UpdateEmployee from './UpdateEmployee';
import DisplayAll from './DisplayAll';
import DeleteEmployee from './DeleteEmployee';
import 'bootstrap/dist/css/bootstrap.min.css';

function App() {
  return (
    <Router>
      <div className="container mt-4 text-primary">
        <h1>Employee Management System</h1>
        <nav className="nav nav-tabs">
          <Link className="nav-link" to="/">Home</Link>
          <Link className="nav-link" to="/add">Add Employee</Link>
          <Link className="nav-link" to="/search">Search Employee</Link>
          <Link className="nav-link" to="/update">Update Employee</Link>
          <Link className="nav-link" to="/display">Display Employee</Link>
          <Link className="nav-link" to="/delete">Delete Employee</Link>
        </nav>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/add" element={<AddEmployee />} />
          <Route path="/search" element={<SearchEmployee />} />
          <Route path="/update" element={<UpdateEmployee />} />
          <Route path="/display" element={<DisplayAll />} />
          <Route path="/delete" element={<DeleteEmployee />} />
        </Routes>
      </div>
    </Router>
  );
}

function Home() {
  return (
    <div className="mt-3">
      <h3>Welcome to the Employee Management System</h3>
      <p>Use the navigation tabs to add, search, or update employee data.</p>
    </div>
  );
}

export default App;
```

---

### `frontend/src/DeleteEmployee.js`

```js
import React, { useState } from "react";
import axios from 'axios';

export default function DeleteEmployee() {
  const [empID, setEmpID] = useState("");
  const [message, setMessage] = useState('');

  const remove = async () => {
    setMessage('');
    if(!empID){ setMessage('Enter EmpID'); return; }
    try {
      await axios.delete(`http://localhost:5000/employees/${empID}`);
      setMessage('Employee deleted successfully');
      setEmpID('');
    } catch (err) {
      setMessage(err.response?.data?.error || 'Error deleting');
    }
  };

  return (
    <div className="mb-3">
      <h3>Delete Employee</h3>
      {message && <div className="alert alert-info">{message}</div>}
      <input value={empID} className="form-control" onChange={e => setEmpID(e.target.value)} placeholder="Enter EmpID" />
      <button className="btn btn-primary mt-3" onClick={remove}>Delete</button>
    </div>
  );
}
```

---

### `frontend/src/DisplayAll.js`

```js
import React, { useEffect, useState } from 'react';
import axios from 'axios';
const API = 'http://localhost:5000';

export default function DisplayAll(){
  const [Employee, setEmployee] = useState([]);
  const [message, setMessage] = useState('');

  const fetchAll = async () => {
    try {
      const res = await axios.get(`${API}/employees`);
      setEmployee(res.data);
    } catch (err) {
      setMessage('Error fetching Employee');
    }
  };

  useEffect(() => { fetchAll(); }, []);

  return (
    <div className="mb-3">
      <h2>All Employee</h2>
      {message && <div>{message}</div>}
      <textarea
        readOnly
        rows={15}
        style={{ width: '100%' }}
        value={Employee.map(b => 
          `EmployeeName: ${b.EmployeeName}\nEmployeeID: ${b.EmployeeID}\nDesignation: ${b.Designation}\nDepartment: ${b.Department}\nJoiningDate: ${b.JoiningDate}\n\n`
        ).join('')}
      />
      <button className="btn btn-primary mt-3" onClick={fetchAll}>Refresh</button>
    </div>
  );
}
```

---

### `frontend/src/SearchEmployee.js`

```js
import React, { useState } from 'react';
import axios from 'axios';

function SearchEmployee() {
  const [EmployeeID, setEmployeeID] = useState('');
  const [result, setResult] = useState('');

  const handleSearch = async () => {
    if (!EmployeeID) {
      setResult('Please enter an EmployeeID');
      return;
    }
    try {
      const res = await axios.get(`http://localhost:5000/employees/${EmployeeID}`);
      const emp = res.data;
      setResult(`
        EmployeeName: ${emp.EmployeeName}
        EmployeeID: ${emp.EmployeeID}
        Designation: ${emp.Designation}
        Department: ${emp.Department}
        JoiningDate: ${new Date(emp.JoiningDate).toLocaleDateString()}
      `);
    } catch (err) {
      setResult(err.response?.data?.error || 'Employee not found');
    }
  };

  return (
    <div className="mt-3">
      <h3>Search Employee</h3>
      <input type="text" className="form-control mb-2"
        placeholder="Enter EmployeeID"
        value={EmployeeID}
        onChange={e => setEmployeeID(e.target.value)} />
      <button className="btn btn-primary mb-3" onClick={handleSearch}>Search</button>
      <textarea className="form-control" rows="8" readOnly value={result} />
    </div>
  );
}

export default SearchEmployee;
```

---

### `frontend/src/UpdateEmployee.js`

```js
import React, { useState } from 'react';
import axios from 'axios';

function UpdateEmployee() {
  const [EmployeeID, setEmployeeID] = useState('');
  const [Designation, setDesignation] = useState('');
  const [message, setMessage] = useState('');

  const handleUpdate = async e => {
    e.preventDefault();
    if (!EmployeeID || !Designation) {
      setMessage('Please fill both fields');
      return;
    }
    try {
      await axios.put(`http://localhost:5000/employees/${EmployeeID}`, { Designation });
      setMessage('Designation updated successfully!');
      setEmployeeID('');
      setDesignation('');
    } catch (err) {
      setMessage(err.response?.data?.error || 'Error updating designation');
    }
  };

  return (
    <div className="mt-3">
      <h3>Update Employee Designation</h3>
      {message && <div className="alert alert-info">{message}</div>}
      <form onSubmit={handleUpdate}>
        <div className="mb-3">
          <label>EmployeeID</label>
          <input type="text" className="form-control"
            value={EmployeeID}
            onChange={e => setEmployeeID(e.target.value)} />
        </div>
        <div className="mb-3">
          <label>New Designation</label>
          <input type="text" className="form-control"
            value={Designation}
            onChange={e => setDesignation(e.target.value)} />
        </div>
        <button className="btn btn-primary" type="submit">Update Designation</button>
      </form>
    </div>
  );
}

export default UpdateEmployee;
```

---

## 🧠 Notes

* MongoDB must be running locally (`mongodb://127.0.0.1:27017/Employees`).
* Both backend (`npm start`) and frontend (`npm start`) should be running simultaneously.
* Make sure CORS is enabled (already handled in code).

---

## 📄 License

This project is open-source under the **MIT License**.

```

---

Would you like me to include a **sample `AddEmployee.js`** code (to complete the CRUD set)?
```
