# Operation_and_Metric_Analytics

create table job_data (
	ds date,
	job_id int,
	actor_id int,
	event varchar,
	language varchar,
	time_spent int,
	org char);
select * from job_data;
insert into job_data (ds) values ('2020-11-30', '2020-11-30', '2020-11-29', '2020-11-28', '2020-11-28', '2020-11-27', '2020-11-26', '2020-11-25');

insert into job_data (ds, job_id, actor_id, event, language, time_spent, org)
values ('2020-11-30', '21','1001', 'skip', 'English', '15', 'A');
insert into job_data values ('2020-11-30', '22', '1006', 'transfer', 'Arabic', '25', 'B');
insert into job_data values ('2020-11-29', '23', '1003', 'decision', 'Persian', '20', 'C');
insert into job_data values ('2020-11-28', '23', '1005', 'transfer', 'Persian', '22', 'D');
insert into job_data values ('2020-11-28', '25', '1002', 'decision', 'Hindi', '11', 'B');
insert into job_data values ('2020-11-27', '11', '1007', 'decision', 'French', '104', 'D');
insert into job_data values ('2020-11-26', '23', '1004', 'skip', 'Persian', '56', 'A');
insert into job_data values ('2020-11-25', '20', '1003', 'transfer', 'Italian', '45', 'C');
select * from job_data;
select * from email_events;
select * from events;
select * from users;
-- Case Study1 (Job data) 
-- 1)Number of jobs reviewed: Amount of jobs reviewed over time.
-- Your task: Calculate the number of jobs reviewed per hour per day for November 2020?
select ds,
count (job_id) as reviewed_per_day,
count(job_id)/24 :: decimal(5,1) as jobsreviewed_per_hour
from job_data
where ds between '2020-11-01' and '2020-11-30'
group by ds;
1-- The number of jobs reviewed on November 28th and 30th was twice the average of other days in the month.

2)-- Throughput: It is the no. of events happening per second.
-- Your task: Let’s say the above metric is called throughput. Calculate 7 day rolling average of throughput? For throughput, do you prefer daily metric or 7-day rolling and why?
select ds, No_of_events,
avg(No_of_events) over (order by ds rows between 6 preceding
and current row) as seven_dayrolling_avg
from
(select ds,
count(distinct event) as No_of_events
from job_data
group by ds
order by ds) a;
-- 2-To capture the overarching pattern while minimizing fluctuations, opting for a 7-day rolling average proves advantageous. By smoothing out short-term variations, this approach effectively portrays the long-term trend.

-- 3)Percentage share of each language: Share of each language for different contents.
-- Your task: Calculate the percentage share of each language in the last 30 days?
select language, count(language) as no_of_language,
count(*)*100/sum(count(*)) over () as percentage
from job_data
where ds between '2020-11-01' and '2020-11-30'
group by language;

select language, count(language) as no_of_language,
((count(language)/sum(count(language)) over ())*100,'%') as percentage
from job_data
where ds between '2020-11-01' and '2020-11-30'
group by language;
-- 3-Persian language dominates with a substantial share of 37.5%, while other languages maintain a consistent distribution.

-- 4)Duplicate rows: Rows that have the same value present in them.
-- Your task: Let’s say you see some duplicate rows in the data. How will you display duplicates from the table?
select *, count(*)
from job_data
group by ds, job_id, actor_id, event, language, time_spent, org
having count(*)>1;
-- 4-There are no duplicate rows in the data.

-- Case Study 2 (Investigating metric spike)
-- 5)User Engagement: To measure the activeness of a user. Measuring if the user finds quality in a product/service.
-- Your task: Calculate the weekly user engagement?
select
	extract(week from occured_at) as week_no,
	count(distinct user_id) as user_engagement
from events
where event_type = 'engagement'
group by week_no; 
-- 5-The highest engagement of users in the week 31 that is 1443 and the lowest engagement of users in the week 18 that is 701    

-- 6)User Growth: Amount of users growing over time for a product.
-- Your task: Calculate the user growth for product?
select year, week, no_of_users,
sum(no_of_users) over (order by year, week ) as cumulative_users
from (
	select 
	extract(year from created_at) as year,
	extract(week from created_at) as week,								
	count(distinct user_id) as no_of_users from users
	group by year, week) as a
	order by no_of_users;
-- ans6)The 35th week of 2014 saw the greatest no of users actively engaging with the product or service while the 2nd week of 2013 had the lowest no of active users

-- 7)Weekly Retention: Users getting retained weekly after signing-up for a product.
-- Your task: Calculate the weekly retention of users-sign up cohort?
select *,
concat(((weekly_users/lag(weekly_users)
		 over (order by week))*100),'%') as 
		 retention_based_on_previous_week
from (select extract(week from occured_at) as week,
	count(distinct user_id) as weekly_users
from events
	where event_name = 'login'
	group by week) a;
-- 7-calculated by dividing the number of retained users by the initial number of users in the cohort and multiplying by 100 to get a percentage. 	
--	
SELECT *,
  concat(((weekly_users/lag(weekly_users) OVER (ORDER BY week))*100), '%') AS "retention based on previous week"
FROM
  (SELECT extract(week from occured_at) AS week,
    count(distinct user_id) AS weekly_users
  FROM events
  WHERE event_name = 'login'
  GROUP BY week) a;
-- 8)Weekly Engagement: To measure the activeness of a user. Measuring if the user finds quality in a product/service weekly.
-- Your task: Calculate the weekly engagement per device?
select 
	extract(week from occured_at) as week,
	device,
	count(distinct user_id) as users_engagement
from events
group by device, week
order by week;
-- 8-Week after week, the MacBook Pro emerges as the device of choice, with users consistently gravitating towards its utilization.

-- 9)Email Engagement: Users engaging with the email service.
-- Your task: Calculate the email engagement metrics?
select 
	action, extract(month from occurred_at) as month,
	count(action) as no_of_mails
from email_events
group by action, month
order by month; 
-- 9-August witnessed the peak of weekly digest email distribution, ensuring users received an abundant volume throughout the month.

