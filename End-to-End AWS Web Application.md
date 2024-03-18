# Architect and Build an End-to-End AWS Web Application from Scratch.
In this tutorial, you will perform an end-to-end serverless deployment of an application using AWS Amplify, Lambda, API Gateway, and AWS RDS.

![Screenshot (8)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/8f3aa3f8-f63e-42fb-8c4e-9db73bcff70a)

Services we will be using
# What do we need?

![Screenshot (9)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/c68a45fd-99be-444e-a4a4-a5cdd675c535)

These are what we need to complete this project.
# What do you need?

![Screenshot (10)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/17382776-96bb-45e5-a536-524feda4a295)
# A. DEPLOY AND HOST A WEBPAGE WITH AWS AMPLIFY
1. Sign in to your AWS account.
2. Search for AWS Amplify in the search bar
![image](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/6442103c-9bce-4b6e-8175-7e5fd7d1f545)

   
![Screenshot (11)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/1d1b5c1e-5d89-4555-9678-61e9edcab220)

4. Click on **Get Started** or __New app__ > __Host web app__
5. For the code source, click on Deploy without Git Provider, and then click on Continue
![Screenshot (12)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/6d1506be-b1a9-4fc1-8f26-0e8f08616dc4)

5. Fill out the manual deployment options with the following:
App name: PowerOfMath2
Environment name: dev
Method: Select Drag and drop
![Screenshot (13)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/ed29fdbf-ee8f-4d1e-a543-fbb3bdea20aa)

*Drag and drop* this [index.html.zip](https://github.com/Tch-22zero5/AWS-Projects/files/14590315/index.html.zip)
file. Note that it is a zip file, you can extract it on your local computer to view the index.html file.

6. Click on Save and deploy
7. You will be directed to the AWS Amplify console where the newly deployed app is. If you encounter a white screen, refresh your browser.
![image](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/1136a812-daa6-49d6-9a15-8d4b8216d0f6)

8. Copy the link under Domain, and paste it into a new tab in your browser
# B. CREATE A PYTHON LAMBDA FUNCTION
9. Search for lambda in the search bar
10. Click on the Create function button
11. Configure the Function with the following:
Click on Author from scratch
Function name: PowerOfMathFunction2
Runtime: Python 3.9
Architecture: x86_64
12. Click on the Create function button
13. Click on the newly created lambda function and locate the default
lambda_function.py file.
14. Copy the code in this file and use it to replace the code in the
lambda_function.py file. This code will run the function to return the
exponent of a base number and return the result.
15. Press Ctrl + S to save the file
16. Click on Deploy
17. Now we need to test our deployment. But before we do that, we need
to configure the test event.
18. Click on the right arrow just beside the Test button
19. Click on Configure test event in the dropdown list that appears
20. Configure the test event with the following parameters:
Event Name: PowerOfMathTestEvent2
Event Sharing Settings: Private
*Replace Event JSON with the code below:
Event JSON:
{
“base”: 2,
“exponent”: 3
}
21. Click on Save, then Click on Test.
22. The test will execute the lambda function and return a Response,
Function logs, and the request ID.
# C. CREATING A REST API FOR THE LAMBDA FUNCTION USING API GATEWAY
Now that we have the Lambda function up and running, we need an endpoint to
hit when we need to invoke the Lambda function. To do this, we will be utilizing
API Gateway.
23. Search for API gateway in the search bar
24. Click on Create API
25. On the next page, click on Build in the REST API option.
26. Configure the REST API with the following:
Protocol: REST
Create new API: New API
API Name: PowerOfMathAPI2
Endpoint Type: Regional
Click on Create API
Note: The interface of your API Gateway may vary from that used for this
documentation due to some changes in design by AWS, however, the
functionality is the same.
27. Once the API has been created, you will be directed to a page where you can
further configure the API. In the tab on the left of your screen, click on
Resources. Then click on the / sign.
28. Click on the Actions button, and select Create Method.
# D 29. Create a POST method with the following configuration:
Integration: Lambda Function
Lambda Function: PowerOfMathFunction2
Click on Next
30. Next, we need to enable CORS(Cross Origin Resource Sharing). This will
enable our Amplify DNS or origin to communicate with our API.
Click on the newly created POST method, and in the Actions, click on Enable
CORS.
31. You will be directed to a page to configure CORS. Ensure that the POST
checkbox is ticked, leave the others in their default states, and click on Enable
CORS.
32. Next we need to deploy the API. Click on the / sign, then click on the Action
button, and then click on Deploy API.
33. A modal will then appear, where you will specify the stage name for the API.
Deployment stage: New Stage
Stage name: dev
Click on Deploy
34. An invoke URL will be given. That is the URL to the API. Copy and paste it in
a text editor. You will use it later.
35. Now, we need to test the API. To do that click on the POST method, then
click on Test.
36. In the Request Body of the POST Method test, paste this JSON:
{
“base”: 2,
“exponent”: 4
}
Click on Test
37. The result should look like this
# E. SETTING UP DYNAMO DB INSTANCE TO STORE DATA
38. Search for dynamo db in the search bar
39. Click on Create table
40. Create a table with the following configuration:
Table name: PowerOfMathDatabase2
Partition key: ID
*Leave the rest in their default
Click on Create table
41. Click on the newly created Dynamo DB table, and navigate to Overview >
General Information > Additional Information. Copy the Amazon Resource Name
(ARN) of the table, and paste it in a text editor.
# F. GIVING LAMBDA PERMISSION TO WRITE TO DYNAMO DB TABLE
Here, we will be giving our Lambda function the required permission to write data to the Dynamo DB table.

42. Navigate to your Lambda function.
  
44. Click on the Configuration tab

45. On the sidebar, click on Permissions.

46. Under Execution role, click on the Role name. This will open up a new tab in the IAM console.

47. The IAM console will display the Role for your Lambda function which is automatically created. Navigate to the Permissions tab > Permission policies > Add permission > Create inline policy.

This will take us to the Create Policy wizard. Select JSON.

Delete the default policy, and replace it with the policy in this
[Execution Role Policy JSON.txt](https://github.com/Tch-22zero5/AWS-Projects/files/14635139/Execution.Role.Policy.JSON.txt)

48. Replace the “Resource” value with the arn of your table.

49. Click on Review policy. Name the policy: “PowerOfMathDynamoPolicy”, and then click on Create policy.
# G. UPDATING THE LAMBDA FUNCTION TO WRITE TO THE DYNAMO DB
50. Navigate to Lambda, and click on the lambda function you created. Replace
the content of lambda_function.py with this file. Save the file, deploy, and test.
51. Navigate back to PowerOfMathDatabase2 in Dynamo DB. Click on Explore
table items.
52. This should display the result of the lambda test which has been uploaded to
the table.
[fctctc.pdf](https://github.com/Tch-22zero5/AWS-Projects/files/14635244/fctctc.pdf)
# H. UPDATING THE index.html FILE TO CALL API GATEWAY
3. Download this file [index.html1.txt](https://github.com/Tch-22zero5/AWS-Projects/files/14635322/index.html1.txt) and open it in a text editor.
54. Edit this line, and replace the placeholder with the endpoint of your API Gateway.

// make API call with parameters and use promises to get response
fetch("YOUR API GATEWAY ENDPOINT", requestOptions)
.then(response => response.text())
.then(result => alert(JSON.parse(result).body))
.catch(error => console.log('error', error));

55. Zip the file, drag and drop it in the PowerOfMath2 app on Amplify.
56. Reload the URL of your app and it should appear like this
![Screenshot (16)](https://github.com/Tch-22zero5/AWS-Projects/assets/140101993/7276faf5-f54a-4786-b498-ef0aa52049d8)
57. Test the app by inputting numbers for the calculation, and you should get a prompt that displays the answer.
# DELETING AND CLEANING UP
You will need to delete the resources used in order not to incur charges from AWS.
58. Navigate to your Amplify app. Click on Actions > Delete app. Follow the instructions to complete the delete.
59. Navigate to Dynamo DB, select PowerOfMathDatabase2, and click on Delete. Follow the instructions to complete the delete.
60. Navigate to Lambda > Functions, select PowerOfMathFunction2. Click on Actions > Delete. Follow the instructions to complete the delete.
# CONGRATULATIONS.
