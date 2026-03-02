Departmental Classification Model – Feature Selection Explanation

The goal of this model is to send each complaint to the correct CFPB department. Only information available when the complaint is first submitted is used. Any column that is added later, used for tracking, or not related to department type is removed.

Target
Product – This shows which department handles the complaint. It is what the model is trying to predict.

Feature
Consumer complaint narrative – This is the text written by the customer. It explains the problem and is the main information used to decide the department.

Removed Columns
Date received – Only shows when the complaint was submitted - doesnt determine department
Sub-product – Very similar to Product causes leaking of information.
Issue – Added by CFPB after reviewing the complaint.
Sub-issue – More detailed label added by CFPB.
Company public response – Added after the company replies.
Company – The company name is not needed to decide the department.
State – Location is not used in this model.
ZIP code – Detailed location not needed.
Tags – Special labels like Servicemember or Older American; not needed for department decision.
Consumer consent provided? – Administrative information.
Submitted via – Shows how the complaint was sent (web, phone, etc.). - doesnt determine department
Date sent to company – Happens after routing.
Company response to consumer – Added after company action.
Timely response? – Shows if the company replied on time.
Consumer disputed? – Shows if the consumer disagreed later.
Complaint ID – Just a tracking number.

Final Model
Target: Product
Feature: Consumer complaint narrative

This setup keeps the model simple and realistic by using only the complaint text to decide the correct department.


Target category cleaning

Credit reporting or other personal consumer reports   
- This category includes complaints about problems with a person’s credit report or credit score, such as incorrect information, fraud accounts, or errors that were not properly fixed by the credit bureau (eg. Experian, Equifax, TransUnion). These bureaus gather information from banks, credit card companies, lenders, and collection agencies about things like loans, payments, and debts.

Credit reporting, credit repair services, or other personal consumer reports
- is just a newer, expanded label.Credit repair services are companies that claim they can improve your credit score or remove negative items from your credit report.

#1 Credit Reporting department
*** we can merge the above two categories in to one Credit Reporting department
also add credit reporting 

>>Predict the broad department  credit Reporting
>>Only for complaints predicted as Credit Reporting, run a second model to split into sub-types
##1Credit report errors
##2Identity theft and fraud accounts
##3Dispute process problems
##4Mixed file errors
##5Credit repair service problems
##6Background and tenant screening report issues
##7Credit score problems
##8Credit freeze and fraud alert issues


#2 Debt Collection
About someone trying to collect money from you
Involves collection agencies or debt buyers
Focuses on harassment, wrong amount, lawsuit threats, calling too often, contacting wrong person
// Debt Collection is about problems with a company trying to collect money from a person, while Credit Reporting is about problems with the information shown on a person’s credit report.


#3 Bank Accounts
created by merging Checking or savings account and Bank account or service 
This category covers complaints about regular bank accounts, such as:
Unauthorized withdrawals
Account freezes or closures
Overdraft fees
Problems accessing funds
Incorrect transactions

#4 Mortgage
Mortgage complaints involve home loans and related issues such as:
Problems with monthly payments
Escrow account errors
Loan modification requests
Foreclosure concerns
Interest rate or payment changes
// The target is typically the company managing the home loan, not a credit bureau or debt collector.

#5 Money Transfers and Digital Payments
created by mergin Money transfer, virtual currency, or money service,  Money transfers,  Virtual currency

#6 Credit cards
created by merging Credit card or prepaid card,  Credit card, Prepaid card 

#7 Student Loans

#8 Vehicle loan or lease 

#9 Consumer loan
Payday loan, title loan, or personal loan merged with Payday loan, title loan, personal loan, or advance loan,
consumer loan, Payday loan 

#10 Financial Services Support
merging Other financial service and  Debt or credit management
// This category includes complaints about companies that provide financial help or support services but are not direct lenders, banks, credit bureaus, or debt collectors. It covers issues with debt settlement companies, credit counseling services, and other financial service providers that help manage or reduce debt. Complaints may involve high fees, unclear terms, failed promises, or poor service.
