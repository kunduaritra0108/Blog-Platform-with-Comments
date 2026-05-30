const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const cors = require('cors');
const db = require('./database');

const app = express();
app.use(express.json());
app.use(cors());

const SECRET_KEY = "your_super_secret_key"; // In production, use environment variables

// --- AUTHENTICATION MIDDLEWARE ---
const authenticateToken = (req, res, next) => {
  const token = req.headers['authorization'];
  if (!token) return res.status(401).json({ error: "Access denied" });

  jwt.verify(token.split(" ")[1], SECRET_KEY, (err, user) => {
    if (err) return res.status(403).json({ error: "Invalid token" });
    req.user = user;
    next();
  });
};

// --- API ENDPOINTS ---

// 1. User Registration
app.post('/api/register', (req, res) => {
  const { username, password } = req.body;
  const hashedPassword = bcrypt.hashSync(password, 10);
  
  db.run(`INSERT INTO users (username, password) VALUES (?, ?)`, [username, hashedPassword], function(err) {
    if (err) return res.status(400).json({ error: "Username already exists" });
    res.json({ message: "User registered successfully", userId: this.lastID });
  });
});

// 2. User Login
app.post('/api/login', (req, res) => {
  const { username, password } = req.body;
  
  db.get(`SELECT * FROM users WHERE username = ?`, [username], (err, user) => {
    if (!user || !bcrypt.compareSync(password, user.password)) {
      return res.status(401).json({ error: "Invalid credentials" });
    }
    const token = jwt.sign({ id: user.id, username: user.username }, SECRET_KEY, { expiresIn: '1h' });
    res.json({ message: "Login successful", token });
  });
});

// 3. Create a Blog Post
app.post('/api/posts', authenticateToken, (req, res) => {
  const { title, content } = req.body;
  db.run(`INSERT INTO posts (title, content, user_id) VALUES (?, ?, ?)`, [title, content, req.user.id], function(err) {
    if (err) return res.status(500).json({ error: err.message });
    res.json({ message: "Post created", postId: this.lastID });
  });
});

// 4. Get All Blog Posts
app.get('/api/posts', (req, res) => {
  db.all(`SELECT posts.id, title, content, username FROM posts JOIN users ON posts.user_id = users.id`, [], (err, rows) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(rows);
  });
});

// 5. Update a Blog Post
app.put('/api/posts/:id', authenticateToken, (req, res) => {
  const { title, content } = req.body;
  db.run(`UPDATE posts SET title = ?, content = ? WHERE id = ? AND user_id = ?`, 
  [title, content, req.params.id, req.user.id], function(err) {
    if (this.changes === 0) return res.status(403).json({ error: "Unauthorized or post not found" });
    res.json({ message: "Post updated successfully" });
  });
});

// 6. Delete a Blog Post
app.delete('/api/posts/:id', authenticateToken, (req, res) => {
  db.run(`DELETE FROM posts WHERE id = ? AND user_id = ?`, [req.params.id, req.user.id], function(err) {
    if (this.changes === 0) return res.status(403).json({ error: "Unauthorized or post not found" });
    res.json({ message: "Post deleted successfully" });
  });
});

// 7. Add a Comment to a Post
app.post('/api/posts/:id/comments', authenticateToken, (req, res) => {
  const { comment } = req.body;
  db.run(`INSERT INTO comments (post_id, user_id, comment) VALUES (?, ?, ?)`, 
  [req.params.id, req.user.id, comment], function(err) {
    if (err) return res.status(500).json({ error: err.message });
    res.json({ message: "Comment added", commentId: this.lastID });
  });
});

// 8. Get Comments for a Post
app.get('/api/posts/:id/comments', (req, res) => {
  db.all(`SELECT comments.id, comment, username FROM comments JOIN users ON comments.user_id = users.id WHERE post_id = ?`, 
  [req.params.id], (err, rows) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(rows);
  });
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
