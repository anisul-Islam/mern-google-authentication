# google-authentication
- [doc](https://blog.logrocket.com/guide-adding-google-login-react-app/)
- front end site
```js
import React, { useEffect } from "react";
import GoogleLogin from "react-google-login";
import { gapi } from "gapi-script";
import { loginWithGoogle } from "../services/AuthService";

const Google = () => {
  const responseGoogle = async (response) => {
    try {
      // 1. get the tokenId from response
      console.log(response.tokenId);

      // 2. send the tokenId to the server
      const result = await loginWithGoogle({ idToken: response.tokenId });
      console.log("Google signin success: ", result);
    } catch (error) {
      console.log("Google signin error: ", error.response.data.message);
    }
  };

  useEffect(() => {
    function start() {
      gapi.client.init({
        clientId: process.env.REACT_APP_CLIENT_ID,
        scope: "email",
      });
    }

    gapi.load("client:auth2", start);
  }, []);

  return (
    <div className="pb-3">
      <GoogleLogin
        clientId={`${process.env.REACT_APP_CLIENT_ID}`}
        render={(renderProps) => (
          <button
            onClick={renderProps.onClick}
            disabled={renderProps.disabled}
            className="btn btn-danger btn-lg btn-block"
          >
            <i className="fab fa-google p-2"></i>Login With Google
          </button>
        )}
        onSuccess={responseGoogle}
        onFailure={responseGoogle}
        cookiePolicy={"single_host_origin"}
      />
    </div>
  );
};

export default Google;

```

```js
export const loginWithGoogle = async (data) => {
  const response = await axios.post(
    `http://localhost:3030/api/google-login`,
    data
  );
  return response.data;
};

```
- backend
- install `npm install google-auth-library`
```js
// google
authRoutes.post("/google-login", handleGoogleLogin);

const { OAuth2Client } = require("google-auth-library");
const jwt = require("jsonwebtoken");

const handleGoogleLogin = async (req, res) => {
  try {
    // creating client which help us to verify the idToken that we receive from front end
    const client = new OAuth2Client(dev.app.googleClientId);

    // get the idToken form react app request body
    const { idToken } = req.body;
    console.log("idToken ", idToken);

    // lets verify the idToken
    client
      .verifyIdToken({ idToken, audience: dev.app.googleClientId })
      .then(async (response) => {
      
        console.log("GOOGLE LOGIN RESPONSE", response);
        const { email_verified, name, email } = response.payload;
        
        const exsitingUser = await User.findOne({ email });
        if (email_verified) {
        
          // if the user already exist in our website
          if (exsitingUser) {
            console.log("user exist");
            // create a token for user
            const token = jwt.sign(
              { _id: exsitingUser._id },
              String(dev.app.jwtSecretKey),
              {
                expiresIn: "7d",
              }
            );

            // step 7: create user info
            const userInfo = {
              _id: exsitingUser._id,
              name: exsitingUser.name,
              email: exsitingUser.email,
              phone: exsitingUser.phone,
              isAdmin: exsitingUser.isAdmin,
            };

            return res.json({ token, userInfo });
          } else {
            // if user does not exist create a new user
            let password = email + dev.app.jwtSecretKey;
            const newUser = new User({
              name,
              email,
              password,
            });

            // console.log(newUser);

            const userData = await newUser.save();
            if (!userData) {
              return res.status(400).send({
                message: "user was not created with google",
              });
            }

            // if user is created
            const token = jwt.sign(
              { _id: userData._id },
              String(dev.app.jwtSecretKey),
              {
                expiresIn: "7d",
              }
            );

            // step 7: create user info
            const userInfo = {
              _id: userData._id,
              name: userData.name,
              email: userData.email,
              phone: userData.phone,
              isAdmin: userData.isAdmin,
            };

            return res.json({ token, userInfo });
          }
        } else {
          return res.status(400).send({
            message: "Google login failed try again",
          });
        }
      });
  } catch (error) {
    res.send({
      message: error.message,
    });
  }
};

```
