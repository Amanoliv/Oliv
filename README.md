# Oliv
npm init -y
npm install express mongoose ejs body-parser method-override
/blog-app
│── /views
│   ├── index.ejs
│   ├── post.ejs
│── /public
│   ├── styles.css
│── server.js
const express = require("express");
const mongoose = require("mongoose");
const bodyParser = require("body-parser");
const methodOverride = require("method-override");

const app = express();
app.set("view engine", "ejs");
app.use(express.static("public"));
app.use(bodyParser.urlencoded({ extended: true }));
app.use(methodOverride("_method"));

// Connect to MongoDB
mongoose.connect("mongodb://127.0.0.1:27017/blogDB", { useNewUrlParser: true, useUnifiedTopology: true });

// Blog Schema
const blogSchema = new mongoose.Schema({
    title: String,
    content: String,
    likes: { type: Number, default: 0 },
    comments: [{ username: String, comment: String }]
});
const Blog = mongoose.model("Blog", blogSchema);

// Home Page (Show all posts)
app.get("/", async (req, res) => {
    const posts = await Blog.find();
    res.render("index", { posts });
});

// New Post Page
app.get("/new", (req, res) => {
    res.render("new");
});

// Create Post
app.post("/posts", async (req, res) => {
    await Blog.create({ title: req.body.title, content: req.body.content });
    res.redirect("/");
});

// Show Single Post
app.get("/posts/:id", async (req, res) => {
    const post = await Blog.findById(req.params.id);
    res.render("post", { post });
});

// Like a Post
app.post("/posts/:id/like", async (req, res) => {
    await Blog.findByIdAndUpdate(req.params.id, { $inc: { likes: 1 } });
    res.redirect("back");
});

// Add Comment
app.post("/posts/:id/comment", async (req, res) => {
    const { username, comment } = req.body;
    await Blog.findByIdAndUpdate(req.params.id, { $push: { comments: { username, comment } } });
    res.redirect("back");
});

// Delete Post
app.delete("/posts/:id", async (req, res) => {
    await Blog.findByIdAndDelete(req.params.id);
    res.redirect("/");
});

// Start Server
app.listen(3000, () => console.log("Server running on port 3000"));
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Blog</title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <h1>My Blog</h1>
    <a href="/new">Write a New Post</a>
    <ul>
        <% posts.forEach(post => { %>
            <li>
                <h2><a href="/posts/<%= post._id %>"><%= post.title %></a></h2>
                <p><%= post.content.substring(0, 100) %>...</p>
                <form action="/posts/<%= post._id %>/like" method="POST">
                    <button type="submit">Like (<%= post.likes %>)</button>
                </form>
            </li>
        <% }); %>
    </ul>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <title><%= post.title %></title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <h1><%= post.title %></h1>
    <p><%= post.content %></p>
    <form action="/posts/<%= post._id %>/like" method="POST">
        <button type="submit">Like (<%= post.likes %>)</button>
    </form>

    <h3>Comments:</h3>
    <ul>
        <% post.comments.forEach(comment => { %>
            <li><strong><%= comment.username %></strong>: <%= comment.comment %></li>
        <% }); %>
    </ul>

    <form action="/posts/<%= post._id %>/comment" method="POST">
        <input type="text" name="username" placeholder="Your Name" required>
        <input type="text" name="comment" placeholder="Your Comment" required>
        <button type="submit">Add Comment</button>
    </form>

    <form action="/posts/<%= post._id %>?_method=DELETE" method="POST">
        <button type="submit">Delete Post</button>
    </form>

    <a href="/">Back</a>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <title>New Post</title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <h1>Write a New Post</h1>
    <form action="/posts" method="POST">
        <input type="text" name="title" placeholder="Post Title" required>
        <textarea name="content" placeholder="Post Content" required></textarea>
        <button type="submit">Publish</button>
    </form>
    <a href="/">Back</a>
</body>
</html>
mongod  # Start MongoDB
node server.js  # Start server
