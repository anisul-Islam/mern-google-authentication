# google-authentication
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
