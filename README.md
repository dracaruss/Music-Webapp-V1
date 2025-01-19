# Serverless Web App - RemixKing.net

This is the link to the development work in progress live app: https://d1dlrsjxqn5nwj.cloudfront.net/index.html

Russell here again. This time I'll show the steps I'm taking to create and secure my Serverless AWS Web App.

The end result is to get my web app allowing only authenticated users access to upload music files (only) to my backend data bucket.
The authentication is done through Cognito, and the app utilizes Api-Gateway, IAM, Lambda, HTML/Javascript and S3 Buckets services also to achieve the working end result.

I will do different versions of this project, as it's being improved on daily. 
In this initial setup the JWT token will be exposed (unsafely) in the URL. Though I know its unsafe, I want to run through all scenarios to develop my experience with building and fixing different issues.

##
![00 webpage](https://github.com/user-attachments/assets/d0f65c26-cf35-40fb-bb00-256866c2a6a0)
##

First I setup 2 S3 buckets, one for the front end to be publicly accessible, and the other for the protected back end.

![0 s3 buckets](https://github.com/user-attachments/assets/c3365b0e-abde-4ced-9739-c3425e8fcffe)

Next I configured them for web hosting.

![2 setup static website hosting s3](https://github.com/user-attachments/assets/da31910c-f003-4b93-b4b1-fc1e016891ec)

After I setup my cognito user pool for authentication

![1 1 setup user pool](https://github.com/user-attachments/assets/bca6f71a-974a-4e3d-895a-5b621acab997)

While in Cognito I setup my Identity Pool, and connected it to be authorized from my User Pool.

![3 identity pool](https://github.com/user-attachments/assets/2c7b7c9e-e0af-45df-80a5-31fbc54ff482)

Moving to IAM I prepared the JSON policy for the role that my authenticated users would inherit.

            {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Action": [
                            "s3:GetObject",
                            "s3:PutObject"
                         ],
                         "Effect": "Allow",
                         "Resource": [
                             "arn:aws:s3:::remixking-backend-landingpage/*"
                         ]
                     }
                ]
            }

Now it's time to build my javascript landing page with code that will parse the idToken from the URL.

        async function addUserToIdentityPool(idToken) {
            const client = new AWS.CognitoIdentity({ region: REGION });
            try {
                const response = await client.getId({
                    IdentityPoolId: IDENTITY_POOL_ID,
                    Logins: {
                        [`cognito-idp.${REGION}.amazonaws.com/${USER_POOL_ID}`]: idToken,
                    },
                }).promise();

                const credentials = await client.getCredentialsForIdentity({
                    IdentityId: response.IdentityId,
                    Logins: {
                        [`cognito-idp.${REGION}.amazonaws.com/${USER_POOL_ID}`]: idToken,
                    },
                }).promise();
                
In the public facing bucket I need to load my webpage files, and configure my 'Register' button to redirect to the Cognito Hosted UI I just setup.
##

		  <nav id="navbar-spy" class="navbar-collapse collapse">
			<ul class="nav navbar-nav navbar-right">
			  <li><a href="#slides">HOME</a></li>
			  <li><a href="#studio">STUDIO</a></li>
			  <li><a href="#services">SERVICES</a></li>
			  <li><a href="#gallery">MUSIC</a></li>
			  <li><a href="#team">TEAM</a></li>
			  <li><a href="https://remixking-backend-landingpage.s3.us-east-1.amazonaws.com/upload_page2025_4.html">UPLOAD SONG</a></li>
			  <li><a href="#contact">CONTACT</a></li>
			  <li>
				<a href="#" id="registerButton" class="btn btn-primary">Register</a>
				<script>
				  document.getElementById('registerButton').addEventListener('click', function(event) {
					event.preventDefault(); // Prevent the default action of the link

Once 'Register' is clicked by users, they will be redirected to sign up via Cognito.

![6 sign up via cognito hosted ui page](https://github.com/user-attachments/assets/db5d0aaf-04e9-4946-8903-3ccdd6331760)

After signing up and authenticating, Cognito redirects users to my JS page which takes the idToken in URL (I know not secure) and uses it to create an authenticated user in identity pool.

        async function addUserToIdentityPool(idToken) {
            const client = new AWS.CognitoIdentity({ region: REGION });
            try {
                const response = await client.getId({
                    IdentityPoolId: IDENTITY_POOL_ID,
                    Logins: {
                        [`cognito-idp.${REGION}.amazonaws.com/${USER_POOL_ID}`]: idToken,
                    },
                }).promise();

##

Ok I took a quick diversion to setup my API Gateway before I go any further:

Even though I won't use the API Gateway in this first version of the app, I still set it up now since it's convenient while I was working with the idToken. First I created the API with CORS configuration, and setup the methods.

![8 create api gateway with CORS and GET](https://github.com/user-attachments/assets/6df7a32f-3e52-4927-ad41-7c89d97fe735)

I also created a Lambda function to be invoked by the GET method, for the future.

![9 create lambda](https://github.com/user-attachments/assets/7661501d-df06-43bf-b8a1-0fa0ad0820f4)

Also I gave the Lambda function permissions over the S3 buckets.

![11 give lambda permissions to access S3 and function](https://github.com/user-attachments/assets/be15225f-9930-4e7a-b530-d0f4ea95a6b0)

I needed to authenticate access to the API, so I used the API Cognito Authorizer.

![12 create authorizer for API from Cognito](https://github.com/user-attachments/assets/2a701ec1-b503-4ecf-a493-3d80bd239cbe)

And lastly attached the Authorizer to the GET method. Ok great I'll touch back to the API Gateway in a bit.

![13 attach the authorizer to the method request of the GET for api gateway](https://github.com/user-attachments/assets/904b1c73-93f3-400e-a34f-d401e907914c)

##

Ok back to business, once the idToken creates the Identity user with the temporary privileges, I redirected users to the business end of the app, the actual 'Upload' function. The javascript of this page grabs a file, and validates that it's a music file. I'll also add some more server side validation in the future to sanitize the input further.

        <div class="upload-section">
            <label for="fileInput" class="upload-label">
                <i class="fas fa-cloud-upload-alt"></i> Choose a file
            </label>
            <input type="file" id="fileInput" accept="audio/mpeg,audio/wav">
            <button type="submit" id="uploadButton" class="upload-button">Upload Song</button>
            <div id="status"></div>
            <div class="upload-info">Only MP3 and WAV files are supported</div>
        </div>

Now users can load their song to the backend S3 bucket to a 'songs' folder.

            // Use Fetch API to send the file to S3
            fetch('https://remixking-upload-bucket.s3.us-east-1.amazonaws.com/songs/' + file.name, {
                method: 'PUT', // Use PUT method for direct file upload
                body: file // Send the file as the body
            })
            .then(response => {
                if (response.ok) {
                    status.textContent = "File uploaded successfully!";
                    status.className = 'success';
                } else {
                    status.textContent = "Failed to upload file.";
                    status.className = 'error';
                }
            })
            .catch(error => {
                status.textContent = "Error: " + error.message;
                status.className = 'error';
            });

The upload function is now ready and working!

![19 upload page is now served](https://github.com/user-attachments/assets/9bd3ae06-a321-40b4-80e8-31057ca590bc)

##

The last step was to front the entire web page with CloudFront, and set it up to only allow access to it using OAC.

![22 setup OAC on cloudfront](https://github.com/user-attachments/assets/0e4aa099-0ede-48af-9bf5-579bff1a5bd2)

And for additional App protection, I turned on the CloudFront WAF.

![23 enable WAF on cloudfront](https://github.com/user-attachments/assets/a0fe7fd5-dbd3-446d-b978-d44fe4a0bc83)

The very last step was to enable CORS on the backend bucket.

![21 set backend S3 CORS policy](https://github.com/user-attachments/assets/06a5ac57-0c61-4de7-8e42-da1c4f959588)

And also to setup the bucket policy on the front end S3 bucket to only allow access from CloudFront.

![24 change user pool redirect page to point to cloudfront now](https://github.com/user-attachments/assets/a0412281-66ad-4939-aa1f-d922b1a8836a)

##

Ok great, all up and running, but insecure 😭 because the idToken is in the URL. 

Fear not! Version 2 of the App will fix all of that!
In the next version I will use OAuth 2.0 with code validation instead of implicit grant, to hide the idToken in the Authorization header.

##

Stay tuned! Russell.








