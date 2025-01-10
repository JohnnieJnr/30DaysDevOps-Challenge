# Game Day Notifications / Sports Alerts System

## **Project Overview**
This project is an alert system that sends instant NBA game day score notifications to subscribed users via Email. It leverages AWS services such as **AWS SNS**, **AWS Lambda and Python**, **Amazon EvenBridge** and **SportsdataIO APIs(NBA APIs)** to provide sports fans with up-to-date game information. The project demonstrates event-driven cloud computing principles and efficient notification systems.

---

## **Technical Architecture**
![nba_architecture](https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/504104d8f4a40b15fbfb5152f71f7922402da6a0/Day02/src/assets/img/architecture.png)

## **Features**
- Fetches live NBA game scores using an external API.
- Sends formatted score updates to subscribers via Email using Amazon SNS.
- Scheduled automation for regular updates using Amazon EventBridge.
- Designed with security in mind, following the principle of least privilege for IAM roles.

## **Prerequisites**
- AN account with API Key at [sportsdata.io](https://sportsdata.io/)
- AWS account with basic understanding of AWS and Python

## **Tech Used**
- Cloud Provider: AWS
- Cloud Services: EventBridge, Lambda, SNS, IAM
- Language: Python
- API: SportsDataIO(NBA Game API)

---
## **Setup Instructions**

## **1. Create AWS SNS Topic and Subcription**
- Navigate to the AWS SNS dashboard in the AWS management Console. On the left navigaation panel, click on <b>'Topics'</b> and click on <b>'Create topic'</b> respectively.  

    ./src/assets/img/topic-creation.png

- Choose standard topic type and give the topice a name(eg. game-day-topic) and click on create.

    ./src/assets/img/topic-creation2.png

- After creating the topic, an ARN(Amazon Resource Name) will be assigned to your topic, which you'll need in configuring the Lambda function. After confirming the ARN proceed to create a subcription by clickin on the create subscription button on the right side of the screen.

    ./src/assets/img/topic-arn.png

- Select email from the <b>'Protocol'</b> section, type your email and the <b>'Endpoint'</b> section and click <b>'Create subscription'</b>.

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/subcriptioni-creation.png

- Confirm your email subcription by clickin the link sent to your email 

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/sub-email-confirm.png

## **2. Create IAM Role and Policie**
Next step is to create an AWS Policy and IAM Role for the Lambda function to be able to publish to the SNS topic.

- Navigate to the IAM Console and click on <b>Policies</b>. On the policies dashboard, select <b>SNS</b> as the <b>Service</b>. Use the below policy and click on <b>Next</b>.
    
     ```
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": "sns:Publish",
                    "Resource": "<paste-your-sns-topic-arn-here>"
                }
            ]
        }
    ```
    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/policy-creation.png

- After the creation of the IAM Policy, proceed to create and IAM Role for to be used by the Lambda function. Select both the policy creted and <b>AWSLambdaBasicFunctionRole</b>, click next and give the role a name.

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/iam-role-creation.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/iam-role-creation2.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/iam-role-creation3.png


## **3. Create AWS Lambda Fuction**
AWS Lambda is a serverless compute service that runs your code in response to events and automatically manages the underlying compute resources for you. To trigger the notificaton, we'll craete a lambda function and use it to send messagess to the SNS topic created ealier.

- Navigate to the <b>AWS Lambda dashboard</b> and click the <b>“Create function”</b>.
- Choose the <b>“Author from scratch”</b> option, gve the function a name and select <b>Python</b> as the <b>Runtime</b>. Under the <b>Change default execution role</b>, select <b>Use an existing role</b> and select the role created in the IAM section and click on <b>“Create function”</b>.

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/lambda-creation.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/lambda-creation2.png

- With the Lambda function created, we can add our code to retieve the game-day data from [sportsdata.io](https://sportsdata.io/) and publish it to our SNS topic 

    ```
        import os
        import json
        import urllib.request
        import boto3
        from datetime import datetime, timedelta, timezone

        def format_game_data(game):
            status = game.get("Status", "Unknown")
            away_team = game.get("AwayTeam", "Unknown")
            home_team = game.get("HomeTeam", "Unknown")
            final_score = f"{game.get('AwayTeamScore', 'N/A')}-{game.get('HomeTeamScore', 'N/A')}"
            start_time = game.get("DateTime", "Unknown")
            channel = game.get("Channel", "Unknown")
            
            # Format quarters
            quarters = game.get("Quarters", [])
            quarter_scores = ', '.join([f"Q{q['Number']}: {q.get('AwayScore', 'N/A')}-{q.get('HomeScore', 'N/A')}" for q in quarters])
            
            if status == "Final":
                return (
                    f"Game Status: {status}\n"
                    f"{away_team} vs {home_team}\n"
                    f"Final Score: {final_score}\n"
                    f"Start Time: {start_time}\n"
                    f"Channel: {channel}\n"
                    f"Quarter Scores: {quarter_scores}\n"
                )
            elif status == "InProgress":
                last_play = game.get("LastPlay", "N/A")
                return (
                    f"Game Status: {status}\n"
                    f"{away_team} vs {home_team}\n"
                    f"Current Score: {final_score}\n"
                    f"Last Play: {last_play}\n"
                    f"Channel: {channel}\n"
                )
            elif status == "Scheduled":
                return (
                    f"Game Status: {status}\n"
                    f"{away_team} vs {home_team}\n"
                    f"Start Time: {start_time}\n"
                    f"Channel: {channel}\n"
                )
            else:
                return (
                    f"Game Status: {status}\n"
                    f"{away_team} vs {home_team}\n"
                    f"Details are unavailable at the moment.\n"
                )

        def lambda_handler(event, context):
            # Get environment variables
            api_key = os.getenv("NBA_API_KEY")
            sns_topic_arn = os.getenv("SNS_TOPIC_ARN")
            sns_client = boto3.client("sns")
            
            # Adjust for Central Time (UTC-6)
            utc_now = datetime.now(timezone.utc)
            central_time = utc_now - timedelta(hours=6)  # Central Time is UTC-6
            today_date = central_time.strftime("%Y-%m-%d")
            
            print(f"Fetching games for date: {today_date}")
            
            # Fetch data from the API
            api_url = f"https://api.sportsdata.io/v3/nba/scores/json/GamesByDate/{today_date}?key={api_key}"
            print(today_date)
            
            try:
                with urllib.request.urlopen(api_url) as response:
                    data = json.loads(response.read().decode())
                    print(json.dumps(data, indent=4))  # Debugging: log the raw data
            except Exception as e:
                print(f"Error fetching data from API: {e}")
                return {"statusCode": 500, "body": "Error fetching data"}
            
            # Include all games (final, in-progress, and scheduled)
            messages = [format_game_data(game) for game in data]
            final_message = "\n---\n".join(messages) if messages else "No games available for today."
            
            # Publish to SNS
            try:
                sns_client.publish(
                    TopicArn=sns_topic_arn,
                    Message=final_message,
                    Subject="NBA Game Updates"
                )
                print("Message published to SNS successfully.")
            except Exception as e:
                print(f"Error publishing to SNS: {e}")
                return {"statusCode": 500, "body": "Error publishing to SNS"}
            
            return {"statusCode": 200, "body": "Data processed and sent to SNS"}
     ```

- Go to the <b>“Code”</b> tab, paste code, and click on “Deploy” to deploy the code. In the lambda dashboard, go to configuration and <b>Environment variables</b> and add <b>NBA_API_KEY</b> from your [sportsdata.io](https://sportsdata.io/) account and also the <b>SNS_TOPIC_ARN</b>

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/lambda-code-deploy.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/lambda-code-deploy2.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/lambda-code-deploy3.png

- With the code deployment complete, we can now test to ensure that everything is working as it should. In the lambda function dasshboardl click on <b>Test</b>. If everything is working , you should recieve an email with all the game detail from [sportsdata.io](https://sportsdata.io/)

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/lambda-test.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/lambda-test2.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/lambda-test3.png


## **4. Create EventBridge schedular**
Amazon EventBridge Scheduler is a serverless scheduler that allows you to create, run, and manage tasks from one central, managed service. For this porject we weill use EventBridge to schedule a cront job to trigger the Lambda function to publish the game notification to our SNS topic.

- Navigate to the EventBridge dashboard and click <b>Create role</b>. At the <b> Define rule detail</b> section, give a name to the rule and select <b>Schedule</d> and click <b>Continue in EventBridge Schedule</b>

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-setup.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-setup2.png

- At the <b>Schedule name and description</b> section, give your preferred name to the schedule, select <b>Recurring schedule</b>, <b>Cron-based schedule</b>   

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-schedule.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-schedule2.png

- At the target section, select AWS Lambda as the target and select the Lambda function create to invoke

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-lambda-invoke.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-lambda-invoke2.png

- With that done, we can proceed to finalise the schedule. Next is to leave the rest with the default values, review and create th schedule.

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-schedule-setup.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-schedule-setup2.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-schedule-setup3.png

    https://github.com/JohnnieJnr/30DaysDevOps-Challenge/blob/main/Day02/src/assets/img/EB-schedule-setup4.png

## **Lessons**
- How to design a notification bases system using AWS SNS pub/sub and Lambda.
- Integrating api's with aws lambda.
- Automating workflows using EventBridge cronjob.
