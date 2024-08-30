React:(App.tsx):-
import "./App.css";
import { BrowserRouter, Route, Routes } from "react-router-dom";
import LoginForm from "./login";
import Table from "./table";
import HomePage  from "./home";
import Counter from "./counter";
import Register from "./register";
const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/home" element={<HomePage />} />
        <Route path="/login" element={<LoginForm />} />
        <Route path="/counter" element={<Counter />} />
        <Route path="/table" element={<Table />} />
        <Route path="/register" element={<Register />} />
      </Routes>
    </BrowserRouter>
  );
};
export default App;







(Table.tsx):-
import { useState, useEffect } from "react";
import "./App.css";
import api from "./axios";

export const Table = (props) => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [sidebarOpen, setSidebarOpen] = useState(false);
  const [selectedRow, setSelectedRow] = useState(null);
  const [newName, setNewName] = useState('');
  const [newEmail, setNewEmail] = useState('');
  const [newPhoneNumber, setNewPhoneNumber] = useState('');
  const [newPassword, setNewPassword] = useState('');

  const getData = async () => {
    setLoading(true);
    const response = await api.get("/getdata");
    const responseValue = response.data;
    setLoading(false);
    setData(responseValue);
  };

  const handleEdit = (row) => {
    setSelectedRow(row);
    setNewName(row.name);
    setNewEmail(row.email);
    setNewPhoneNumber(row.Phonenumber);
    setNewPassword(row.password);
    setSidebarOpen(true);
  };

  const handleCloseSidebar = () => {
    setSidebarOpen(false);
    setSelectedRow(null);
  };

  const handleUpdate = async () => {
    try {
      const updatedUser = {
        name: newName,
        email: newEmail,
        Phonenumber: newPhoneNumber,
        password: newPassword
      };
      await api.patch(`/users/${selectedRow._id}`, updatedUser);
      const updatedData = data.map((row) => {
        if (row._id === selectedRow._id) {
          return { ...row, ...updatedUser };
        }
        return row;
      });
      setData(updatedData);
      handleCloseSidebar();
    } catch (error) {
      console.error(error);
    }
  };

  useEffect(() => {
    getData();
  }, []);

  return (
    <>
      <button onClick={() => getData()}>Get Data</button>
      <button onClick={() => setData([])}>Reset</button>
      <table>
        <thead>
          <tr>
            <th>S.No</th>
            <th>User Name</th>
            <th>E-mail</th>
            <th>Phone</th>
            <th>Password</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          {data.map((user, index) => (
            <tr key={index}>
              <td>{index + 1}</td>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <td>{user.Phonenumber}</td>
              <td>{user.password}</td>
              <td>
                <button onClick={() => handleEdit(user)}>Edit</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
      {sidebarOpen && (
        <div
          style={{
            position: "fixed",
            top: 0,
            right: 0,
            width: "300px",
            height: "100vh",
            backgroundColor: "green",
            padding: "20px",
            zIndex: 1000,
            color: "#000"
          }}
        >
          <h2>Edit User</h2>
          <p>
            Username:{" "}
            <input
              type="text"
              value={newName}
              onChange={(e) => setNewName(e.target.value)}
            />
          </p>
          <p>
            Email:{" "}
            <input
              type="email"
              value={newEmail}
              onChange={(e) => setNewEmail(e.target.value)}
            />
          </p>
          <p>
            Phonenumber:{" "}
            <input
              type="tel"
              value={newPhoneNumber}
              onChange={(e) => setNewPhoneNumber(e.target.value)}
            />
          </p>
          <p>
            Password:{" "}
            <input
              type="password"
              value={newPassword}
              onChange={(e) => setNewPassword(e.target.value)}
            />
          </p>
          <button onClick={handleCloseSidebar}>Close</button>
          <button onClick={handleUpdate}>Update</button>
        </div>
      )}
    </>
  );
};
export default Table;





(Register.tsx):-
import { useState } from "react";
import './axios'
import api from "./axios";
const Register = (props) => {
  const [message, setMessage] = useState(false);
  const [data, setData] = useState({ 
    username: "", 
    email: "", 
    phonenumber: "", 
    password: "", 
    confirmPassword: "" 
  });
  const [error, setError] = useState(null);

  const updateData = (type, event) => {
    setData({ ...data, [type]: event.target.value });
  };

  const handleRegister = async (e) => {
    e.preventDefault();
    const response = await api.post("/register",data )
    console.log(data);
    const result = await response.data;
    console.log('result: ', result)
    setMessage("Registration successful!");
    console.log(result);
  };
  return (
    <div style={{ padding: "20px", textAlign: "center" }}>
      <h1>Register Form</h1>
      <div>
        <label>
          Username:
          <input
            type="text"
            value={data.username}
            onChange={(e) => updateData("username", e)}
          />
        </label>
      </div>
      <br />
      <div>
        <label>
          Email:
          <input
            type="email"
            value={data.email}
            onChange={(e) => updateData("email", e)}
          />
        </label>
      </div>
      <br />
      <div>
        <label>
          phonenumber:
          <input
            type="number"
            value={data.phonenumber}
            onChange={(e) => updateData("phonenumber", e)}
          />
        </label>
      </div>
      <br />
      <div>
        <label>
             Password:
          <input
            type="password"
            value={data.password}
            onChange={(e) => updateData("password", e)}  />
        </label>
        </div>
        <br />
        {error && <p style={{ color: "red" }}>{error}</p>}
        {message && <p style={{ color: "green" }}>{message}</p>}
        <button onClick={handleRegister}>Register</button>
    </div>
    );
    };
export default Register;






Backed: MongoDb(python.py):-
from flask import Flask, request, jsonify
from pymongo import MongoClient
from flask_cors import CORS

app = Flask(__name__)

cors = CORS(app, resources={r"/*": {"origins": "*"}})

client = MongoClient('mongodb://localhost:27017/')
mdb = client['pro456']

@app.route("/register", methods=['POST'])
def register():
    input_data = request.get_json()
    name = input_data.get('username')
    email = input_data.get('email')
    password = input_data.get('password')
    phonenumber = input_data.get('phonenumber')
    user = mdb.users.find_one({'email': email})
    if user:
        return jsonify({
            'status': 0,
            'msg': "User already exists",
            'class': "error"
        })
    count = mdb.users.count_documents({})
    new_user = {
        "_id": count + 1,
        "name": name,
        "email": email,
        "password": password,
        "Phonenumber": phonenumber,
    }
    mdb.users.insert_one(new_user)
    return jsonify({"status": 1, "msg": "Inserted successfully"})


@app.route("/getdata", methods=['GET'])
def getData():
    users = list(mdb.users.find({}, {"_id": 1, "name": 1, "email": 1, "Phonenumber": 1, "password": 1}))
    return jsonify(users)


@app.route("/users/<int:user_id>", methods=['PATCH'])
def update_user(user_id):
    input_data = request.get_json()
    updated_fields = {
        "name": input_data.get('name'),
        "email": input_data.get('email'),
        "Phonenumber": input_data.get('Phonenumber'),
        "password": input_data.get('password')
    }
    result = mdb.users.update_one({"_id": user_id}, {"$set": updated_fields})
    if result.modified_count == 1:
        return jsonify({"status": 1, "msg": "User updated successfully"})
    else:
        return jsonify({"status": 0, "msg": "User update failed"})


if __name__ == '__main__':
    app.run(debug=True)





