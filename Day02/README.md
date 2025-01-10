# Game Day Notifications / Sports Alerts System

## **Project Overview**
This project is an alert system that sends instant NBA game day score notifications to subscribed users via Email. It leverages AWS services such as **AWS SNS**, **AWS Lambda and Python**, **Amazon EvenBridge** and **SportsdataIO APIs(NBA APIs)** to provide sports fans with up-to-date game information. The project demonstrates event-driven cloud computing principles and efficient notification systems.

---

## **Technical Architecture**
![nba_API](Day02/src/assets/img/architecture.png)

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

## **Create AWS Topic and Subcription**

## **Create IAM Role and Policie**

## **Create AWS Lambda Fuction**

## **Create EventBridge schedular**


## **Lessons**
- How to design a notification bases system using AWS SNS pub/sub and Lambda.
- Integrating api's with aws lambda.
- Automating workflows using EventBridge cronjob.
