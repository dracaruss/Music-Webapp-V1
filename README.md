# Serverless Web App - RemixKing.net

This is the link to the development work in progress live app: https://d1dlrsjxqn5nwj.cloudfront.net

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

After I setup my cognito user pool for authentication.

![1 setup user pool](https://github.com/user-attachments/assets/a33e314b-7bff-40e8-a99d-88bbcf5e2b1b)

While in Cognito I setup my Identity Pool, and connected it to be authorized from my User Pool.

![3 identity pool](https://github.com/user-attachments/assets/2c7b7c9e-e0af-45df-80a5-31fbc54ff482)

Moving to IAM I prepared the role for my authenticated users.

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

Now it's time to build my javascript landing page that will parse the idToken from the URL.

![4 get idtoken and exchange for identity pool temp aws creds](https://github.com/user-attachments/assets/9a174c10-57b1-40ea-8acf-d4ca94ebe8f6)

In the public facing bucket I need to load my webpage files, and configure my 'Register' button to redirect to the Cognito Hosted UI I just setup.

![5 index points to cognito and backend upload page](https://github.com/user-attachments/assets/9449f6f0-d4ee-4abe-9303-d38f5b67cb0e)

Once 'Register' is clicked by users, they will be redirected to sign up via Cognito.

![6 sign up via cognito hosted ui page](https://github.com/user-attachments/assets/db5d0aaf-04e9-4946-8903-3ccdd6331760)

After signing up and authenticating, Cognito redirects users to my JS page which takes the idToken in URL (I know not secure) and uses it to create an authenticated user in identity pool.

![7 use the idtoken to get identity creds](https://github.com/user-attachments/assets/a0ea7239-6f03-446f-822f-c33f0dbce58d)

##

Ok quick diversion to setup my API Gateway before I go any further:

Even though I won't use the API Gateway in this first version of the app, I'll still set it up now since it's convenient while I'm working with the idToken. First I'll create the API with CORS configured, and setup the methods.

![8 create api gateway with CORS and GET](https://github.com/user-attachments/assets/6df7a32f-3e52-4927-ad41-7c89d97fe735)

I'll also create a Lambda to be invoked by the GET method, for the future.

![9 create lambda](https://github.com/user-attachments/assets/7661501d-df06-43bf-b8a1-0fa0ad0820f4)

Also give the lambda permissions over the S3 buckets.

![11 give lambda permissions to access S3 and function](https://github.com/user-attachments/assets/be15225f-9930-4e7a-b530-d0f4ea95a6b0)

Also authenticate access to the API via the API Cognito Authorizer.

![12 create authorizer for API from Cognito](https://github.com/user-attachments/assets/2a701ec1-b503-4ecf-a493-3d80bd239cbe)

And lastly attach the Authorizer to the GET method, ok great I'll touch back the API Gateway in a bit.

![13 attach the authorizer to the method request of the GET for api gateway](https://github.com/user-attachments/assets/904b1c73-93f3-400e-a34f-d401e907914c)

##

Ok back to business, once the idToken creates the Identity user with the temporary privileges, I redirect users to the business end of the app, the actual 'Upload' function. The javascript of this page grabs a file, and validates that it's a music file. I'll add some server side validation in the future to also sanitize the input.

        <div class="upload-section">
            <label for="fileInput" class="upload-label">
                <i class="fas fa-cloud-upload-alt"></i> Choose a file
            </label>
            <input type="file" id="fileInput" accept="audio/mpeg,audio/wav">
            <button type="submit" id="uploadButton" class="upload-button">Upload Song</button>
            <div id="status"></div>
            <div class="upload-info">Only MP3 and WAV files are supported</div>
        </div>

Then to load the song to the backend bucket to a 'songs' folder.

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

The last step is to front the web page with CloudFront, and set it up to only allow access to it using OAC.

![22 setup OAC on cloudfront](https://github.com/user-attachments/assets/0e4aa099-0ede-48af-9bf5-579bff1a5bd2)

And for additional protection, turn on the CloudFront WAF.

![23 enable WAF on cloudfront](https://github.com/user-attachments/assets/a0fe7fd5-dbd3-446d-b978-d44fe4a0bc83)

The very last step is to enable CORS on the backend bucket.

![21 set backend S3 CORS policy](https://github.com/user-attachments/assets/06a5ac57-0c61-4de7-8e42-da1c4f959588)

And also to setup the policy on the front end S3 bucket to only allow access from CloudFront.

![24 change user pool redirect page to point to cloudfront now](https://github.com/user-attachments/assets/a0412281-66ad-4939-aa1f-d922b1a8836a)








