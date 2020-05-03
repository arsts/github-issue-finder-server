const express = require("express");
const cors = require("cors");
const passport = require("passport");
const GitHubStrategy = require("passport-github").Strategy;
const keys = require("./config");
const chalk = require("chalk");

let user = {};

passport.serializeUser((user, cb) => {
  cb(null, user);
});

passport.deserializeUser((user, cb) => {
  cb(null, user);
});

// Github Strategy
passport.use(
  new GitHubStrategy(
    {
      clientID: keys.GITHUB.clientID,
      clientSecret: keys.GITHUB.clientSecret,
      callbackURL: "http://localhost:5000/auth/github/callback",
    },
    (accessToken, refreshToken, profile, cb) => {
      console.log(chalk.blue(JSON.stringify(profile)));
      user = { ...profile };
      return cb(null, profile);
    }
  )
);

const app = express();
app.use(cors());
app.options("*", cors());
app.use(passport.initialize());

app.get("/auth/github", passport.authenticate("github"));
app.get(
  "/auth/github/callback",
  passport.authenticate("github", (req, res) => {
    res.redirect("/profile");
  })
);

app.get("/user", (req, res) => {
  console.log("getting user data!");
  console.log(user);

  res.send(user);
});

const database = {
  users: [
    {
      id: "123",
      name: "John",
      email: "john@gmail.com",
      password: "cookies",
      joined: new Date(),
    },
    {
      id: "124",
      name: "Sally",
      email: "sally@gmail.com",
      password: "bananas",
      joined: new Date(),
    },
  ],
};

app.get("/", (req, res) => {
  console.log("sending users");

  res.send(database.users);
});

app.get("/auth/logout", (req, res) => {
  console.log("logging out");
  user = {};
  res.redirect("/");
});

const PORT = 5000;
app.listen(PORT, () => {
  console.log("app is runnig on " + PORT);
});
