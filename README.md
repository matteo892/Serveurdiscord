const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    avatar: { type: String, default: "" },
    role: { type: String, enum: ["user", "admin"], default: "user" },
    createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model("User", userSchema);
require("dotenv").config();
const express = require("express");
const cors = require("cors");
const helmet = require("helmet");
const rateLimit = require("express-rate-limit");
const connectDB = require("./config/db");

const app = express();

// Sécurité
app.use(cors());
app.use(helmet());
app.use(express.json());
app.use(rateLimit({ windowMs: 60 * 1000, max: 100 }));

// Connexion DB
connectDB();

// Routes
app.use("/api/auth", require("./routes/authRoutes"));
app.use("/api/posts", require("./routes/postRoutes"));
app.use("/api/comments", require("./routes/commentRoutes"));
app.use("/api/categories", require("./routes/categoryRoutes"));

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Serveur OK sur port ${PORT}`));
import { createContext, useState, useEffect } from "react";
import { api } from "../api";

export const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
    const [user, setUser] = useState(null);

    useEffect(() => {
        const saved = localStorage.getItem("user");
        if (saved) setUser(JSON.parse(saved));
    }, []);

    const login = async (username, password) => {
        const res = await api.post("/auth/login", { username, password });
        localStorage.setItem("token", res.data.token);
        localStorage.setItem("user", JSON.stringify(res.data.user));
        setUser(res.data.user);
    };

    const register = async (username, password) => {
        await api.post("/auth/register", { username, password });
        return login(username, password);
    };

    const logout = () => {
        setUser(null);
        localStorage.removeItem("token");
        localStorage.removeItem("user");
    };

    const isAdmin = () => user?.role === "admin";

    return (
        <AuthContext.Provider value={{ user, login, register, logout, isAdmin }}>
            {children}
        </AuthContext.Provider>
    );
};
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";
import { useContext } from "react";
import { AuthContext } from "./context/AuthContext";
import Navbar from "./components/Navbar";
import Home from "./pages/Home";
import Login from "./pages/Login";
import Register from "./pages/Register";
import Profile from "./pages/Profile";
import Post from "./pages/Post";
import AdminPanel from "./components/AdminPanel";

function ProtectedAdminRoute({ children }) {
    const { user, isAdmin } = useContext(AuthContext);
    
    if (!user || !isAdmin()) {
        return <Navigate to="/" replace />;
    }
    
    return children;
}

export default function App() {
    return (
        <BrowserRouter>
            <Navbar />
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/login" element={<Login />} />
                <Route path="/register" element={<Register />} />
                <Route path="/profile/:username" element={<Profile />} />
                <Route path="/post/:id" element={<Post />} />
                <Route 
                    path="/admin" 
                    element={
                        <ProtectedAdminRoute>
                            <AdminPanel />
                        </ProtectedAdminRoute>
                    } 
                />
            </Routes>
        </BrowserRouter>
    );
}
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    transition: background 0.3s, color 0.3s;
    min-height: 100vh;
}

body.dark { 
    background: linear-gradient(135deg, #0a1628 0%, #1a2332 100%);
    color: #e8eaf0;
}

body.light { 
    background: linear-gradient(135deg, #f0f4ff 0%, #ffffff 100%);
    color: #1a1a2e;
}

/* Navbar */
.navbar {
    background: linear-gradient(135deg, #0d1b2a 0%, #1b263b 100%);
    padding: 15px 40px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 4px 20px rgba(0,0,0,0.3);
    position: sticky;
    top: 0;
    z-index: 100;
    backdrop-filter: blur(10px);
}

.logo {
    font-size: 28px;
    font-weight: 800;
    color: #fff;
    text-decoration: none;
    display: flex;
    align-items: center;
    gap: 10px;
    transition: transform 0.3s;
}

.logo:hover {
    transform: scale(1.05);
}

.logo-icon {
    font-size: 32px;
    animation: float 3s ease-in-out infinite;
}

@keyframes float {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-5px); }
}

.nav-right {
    display: flex;
    align-items: center;
    gap: 15px;
}

.nav-link {
    color: #e8eaf0;
    text-decoration: none;
    padding: 8px 16px;
    border-radius: 8px;
    transition: all 0.3s;
    display: flex;
    align-items: center;
    gap: 6px;
    font-weight: 500;
}

.nav-link:hover {
    background: rgba(255,255,255,0.1);
    transform: translateY(-2px);
}

.nav-link-primary {
    background: linear-gradient(135deg, #4aa8ff 0%, #3b82f6 100%);
    box-shadow: 0 4px 15px rgba(74, 168, 255, 0.3);
}

.nav-link-primary:hover {
    box-shadow: 0 6px 20px rgba(74, 168, 255, 0.5);
}

.admin-link {
    background: linear-gradient(135deg, #ffd700 0%, #ffa500 100%);
    color: #1a1a2e;
    font-weight: 600;
}

.user-link {
    background: rgba(74, 168, 255, 0.15);
}

.admin-badge-nav {
    background: #ffd700;
    color: #1a1a2e;
    padding: 2px 8px;
    border-radius: 4px;
    font-size: 11px;
    font-weight: 700;
}

.logout {
    background: rgba(255, 94, 94, 0.15);
    color: #ff5e5e;
    border: none;
    padding: 8px 16px;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s;
    font-weight: 500;
    display: flex;
    align-items: center;
    gap: 6px;
}

.logout:hover {
    background: rgba(255, 94, 94, 0.25);
    transform: translateY(-2px);
}

.theme-toggle {
    background: rgba(255,255,255,0.1);
    border: none;
    padding: 8px 12px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 20px;
    transition: all 0.3s;
}

.theme-toggle:hover {
    background: rgba(255,255,255,0.2);
    transform: rotate(15deg) scale(1.1);
}

/* Container */
.home, .profile, .single-post, .admin-panel {
    max-width: 800px;
    margin: 30px auto;
    padding: 20px;
}

/* Create Post */
.create-post {
    background: linear-gradient(135deg, #11263d 0%, #1a3a52 100%);
    padding: 30px;
    margin: 20px 0;
    border-radius: 16px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.3);
    border: 1px solid rgba(74, 168, 255, 0.2);
}

.create-post h3 {
    margin-bottom: 20px;
    color: #4aa8ff;
    font-size: 22px;
}

.create-post input,
.create-post textarea,
.create-post select {
    width: 100%;
    margin: 10px 0;
    padding: 12px 16px;
    border-radius: 10px;
    border: 2px solid rgba(74, 168, 255, 0.3);
    background: rgba(255,255,255,0.05);
    color: #e8eaf0;
    font-size: 15px;
    transition: all 0.3s;
}

.create-post input:focus,
.create-post textarea:focus,
.create-post select:focus {
    outline: none;
    border-color: #4aa8ff;
    background: rgba(255,255,255,0.08);
    box-shadow: 0 0 0 4px rgba(74, 168, 255, 0.1);
}

.form-row {
    display: flex;
    gap: 15px;
    margin: 10px 0;
}

.file-input-wrapper input[type="file"] {
    display: none;
}

.file-label {
    display: inline-block;
    padding: 12px 20px;
    background: linear-gradient(135deg, #4aa8ff 0%, #3b82f6 100%);
    color: white;
    border-radius: 10px;
    cursor: pointer;
    transition: all 0.3s;
    font-weight: 500;
}

.file-label:hover {
    box-shadow: 0 4px 15px rgba(74, 168, 255, 0.4);
    transform: translateY(-2px);
}

.image-preview {
    margin: 15px 0;
    position: relative;
}

.image-preview img {
    width: 100%;
    max-height: 300px;
    object-fit: cover;
    border-radius: 10px;
}

.image-preview button {
    position: absolute;
    top: 10px;
    right: 10px;
    background: rgba(255, 94, 94, 0.9);
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 600;
}

.submit-btn {
    width: 100%;
    margin-top: 15px;
    padding: 14px;
    border-radius: 10px;
    cursor: pointer;
    background: linear-gradient(135deg, #4aa8ff 0%, #3b82f6 100%);
    color: white;
    border: none;
    font-size: 16px;
    font-weight: 600;
    transition: all 0.3s;
}

.submit-btn:hover {
    box-shadow: 0 6px 25px rgba(74, 168, 255, 0.4);
    transform: translateY(-2px);
}

/* Category Filter */
.category-filter {
    margin: 30px 0;
}

.category-filter h4 {
    margin-bottom: 15px;
    color: #4aa8ff;
    font-size: 18px;
}

.category-buttons {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
}

.category-buttons button {
    padding: 10px 20px;
    border-radius: 10px;
    border: 2px solid rgba(74, 168, 255, 0.3);
    background: rgba(74, 168, 255, 0.1);
    color: #e8eaf0;
    cursor: pointer;
    transition: all 0.3s;
    font-weight: 500;
}

.category-buttons button:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(74, 168, 255, 0.2);
}

.category-buttons button.active {
    background: rgba(74, 168, 255, 0.3);
    border-color: #4aa8ff;
    box-shadow: 0 4px 15px rgba(74, 168, 255, 0.3);
}

/* Post Card */
.post-card {
    background: linear-gradient(135deg, #11263d 0%, #1a3a52 100%);
    padding: 25px;
    margin: 20px 0;
    border-radius: 16px;
    box-shadow: 0 4px 20px rgba(0,0,0,0.3);
    border: 1px solid rgba(74, 168, 255, 0.15);
    transition: all 0.3s;
}

.post-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
}

.author-info {
    display: flex;
    align-items: center;
    gap: 10px;
}

.author-link {
    color: #4aa8ff;
    text-decoration: none;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 8px;
    transition: color 0.3s;
}

.author-link:hover {
    color: #64b8ff;
}

.admin-badge {
    background: linear-gradient(135deg, #ffd700 0%, #ffa500 100%);
    color: #1a1a2e;
    padding: 4px 10px;
    border-radius: 6px;
    font-size: 12px;
    font-weight: 700;
}

.post-meta {
    display: flex;
    align-items: center;
    gap: 10px;
}

.category {
    background: rgba(74, 168, 255, 0.2);
    padding: 6px 12px;
    border-radius: 8px;
    font-size: 13px;
    font-weight: 500;
    border: 1px solid rgba(74, 168, 255, 0.3);
}

.delete-btn {
    background: rgba(255, 94, 94, 0.2);
    border: none;
    padding: 6px 10px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 16px;
    transition: all 0.3s;
}

.delete-btn:hover {
    background: rgba(255, 94, 94, 0.4);
    transform: scale(1.1);
}

.post-link {
    text-decoration: none;
    color: inherit;
}

.post-card h3 {
    margin: 15px 0;
    font-size: 22px;
    color: #e8eaf0;
}

.post-content {
    color: #b8bcc8;
    line-height: 1.6;
    margin: 15px 0;
}

.post-img {
    width: 100%;
    border-radius: 12px;
    margin-top: 15px;
    box-shadow: 0 4px 15px rgba(0,0,0,0.3);
}

.post-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 20px;
    padding-top: 15px;
    border-top: 1px solid rgba(74, 168, 255, 0.2);
}

.votes {
    display: flex;
    align-items: center;
    gap: 12px;
}

.vote-btn {
    background: none;
    border: none;
    cursor: pointer;
    font-size: 20px;
    transition: all 0.3s;
    padding: 5px 10px;
    border-radius: 6px;
}

.vote-btn:hover {
    transform: scale(1.3);
}

.vote-btn.up {
    color: #4aa8ff;
}

.vote-btn.up:hover {
    background: rgba(74, 168, 255, 0.2);
}

.vote-btn.down {
    color: #ff5e5e;
}

.vote-btn.down:hover {
    background: rgba(255, 94, 94, 0.2);
}

.vote-count {
    font-weight: 700;
    font-size: 18px;
    color: #4aa8ff;
    min-width: 30px;
    text-align: center;
}

.post-date {
    color: #7a8a9a;
    font-size: 13px;
}

/* Comments */
.comments {
    margin-top: 30px;
    padding: 25px;
    background: rgba(17, 38, 61, 0.5);
    border-radius: 12px;
}

.comments h4 {
    margin-bottom: 20px;
    color: #4aa8ff;
    font-size: 20px;
}

.comments-list {
    margin: 20px 0;
}

.comment {
    background: rgba(255,255,255,0.05);
    padding: 15px;
    margin: 10px 0;
    border-radius: 10px;
    border-left: 3px solid #4aa8ff;
}

.comment-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 8px;
}

.admin-badge-small {
    font-size: 12px;
    margin-left: 6px;
}

.delete-comment-btn {
    background: rgba(255, 94, 94, 0.2);
    color: #ff5e5e;
    border: none;
    padding: 4px 8px;
    border-radius: 4px;
    cursor: pointer;
    font-weight: 600;
    transition: all 0.3s;
}

.delete-comment-btn:hover {
    background: rgba(255, 94, 94, 0.4);
}

.comment-content {
    color: #b8bcc8;
    line-height: 1.5;
    margin: 8px 0;
}

.comment-date {
    color: #7a8a9a;
    font-size: 12px;
}

.comment-form {
    display: flex;
    gap: 10px;
    margin-top: 20px;
}

.comment-form input {
    flex: 1;
    padding: 12px;
    border-radius: 8px;
    border: 2px solid rgba(74, 168, 255, 0.3);
    background: rgba(255,255,255,0.05);
    color: #e8eaf0;
}

.comment-form button {
    padding: 12px 24px;
    border-radius: 8px;
    background: linear-gradient(135deg, #4aa8ff 0%, #3b82f6 100%);
    color: white;
    border: none;
    cursor: pointer;
    font-weight: 600;
    transition: all 0.3s;
}

.comment-form button:hover {
    box-shadow: 0 4px 15px rgba(74, 168, 255, 0.4);
}

.login-prompt {
    color: #7a8a9a;
    text-align: center;
    padding: 20px;
}

/* Auth Forms */
.auth-form {
    max-width: 400px;
    margin: 50px auto;
    padding: 40px;
    background: linear-gradient(135deg, #11263d 0%, #1a3a52 100%);
    border-radius: 16px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.3);
}

.auth-form h2 {
    margin-bottom: 30px;
    color: #4aa8ff;
    text-align: center;
}

.auth-form input {
    width: 100%;
    margin: 12px 0;
    padding: 14px;
    border-radius: 10px;
    border: 2px solid rgba(74, 168, 255, 0.3);
    background: rgba(255,255,255,0.05);
    color: #e8eaf0;
    font-size: 15px;
}

.auth-form button {
    width: 100%;
    margin-top: 20px;
    padding: 14px;
    border-radius: 10px;
    background: linear-gradient(135deg, #4aa8ff 0%, #3b82f6 100%);
    color: white;
    border: none;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
}

.auth-form button:hover {
    box-shadow: 0 6px 25px rgba(74, 168, 255, 0.4);
    transform: translateY(-2px);
}

/* Admin Panel */
.admin-panel {
    background: linear-gradient(135deg, #11263d 0%, #1a3a52 100%);
    padding: 30px;
    border-radius: 16px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.3);
}

.admin-panel h2 {
    color: #ffd700;
    margin-bottom: 30px;
    font-size: 28px;
}

.admin-section {
    margin: 30px 0;
    padding: 25px;
    background: rgba(255,255,255,0.05);
    border-radius: 12px;
}

.admin-section h3 {
    color: #4aa8ff;
    margin-bottom: 20px;
    font-size: 20px;
}

.category-form {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.category-form input {
    padding: 12px;
    border-radius: 8px;
    border: 2px solid rgba(74, 168, 255, 0.3);
    background: rgba(255,255,255,0.05);
    color: #e8eaf0;
}

.form-group {
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.form-group label {
    color: #b8bcc8;
    font-size: 14px;
    font-weight: 500;
}

.categories-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 15px;
    margin-top: 20px;
}

.category-item {
    background: rgba(255,255,255,0.05);
    padding: 15px;
    border-radius: 10px;
    display: flex;
    align-items: center;
    gap: 10px;
    transition: all 0.3s;
}

.cat-icon {
    font-size: 24px;
}

.cat-name {
    flex: 1;
    font-weight: 600;
}

.delete-cat-btn {
    background: rgba(255, 94, 94, 0.2);
    border: none;
    padding: 6px 10px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.3s;
}

.delete-cat-btn:hover {
    background: rgba(255, 94, 94, 0.4);
}

/* Loading */
.loading {
    text-align: center;
    padding: 50px;
}

.spinner {
    width: 50px;
    height: 50px;
    margin: 0 auto 20px;
    border: 4px solid rgba(74, 168, 255, 0.3);
    border-top-color: #4aa8ff;
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    to { transform: rotate(360deg); }
}

.no-posts {
    text-align: center;
    padding: 50px;
    color: #7a8a9a;
}
