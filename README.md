<h1>User Submissions Performance Analysis Project</h1>

<p>This project analyzes user engagement and performance data from a submission-based platform. Using PostgreSQL, we process submission patterns, point distributions, and competitive rankings. The analysis utilizes <strong>Common Table Expressions (CTEs)</strong>, <strong>Window Functions</strong>, and <strong>Conditional Aggregations</strong> to transform raw logs into actionable performance metrics.</p>

<h2>1. Database Schema and Infrastructure</h2> <p>The analysis is built upon a structured table that records every user interaction. We utilize <code>SERIAL</code> for primary keys and <code>TIMESTAMP WITH TIME ZONE</code> to maintain temporal accuracy for global users.</p>

<pre> -- Initialize User Submissions Table CREATE TABLE user_submissions ( id SERIAL PRIMARY KEY, user_id BIGINT, question_id INT, points INT, submitted_at TIMESTAMP WITH TIME ZONE, username VARCHAR(50) );

-- Verify Data Ingestion SELECT * FROM user_submissions; </pre>

<h2>2. Exploratory Data Analysis (EDA)</h2>

<p>The following questions address user activity, accuracy, and competitive rankings.</p>

<h3>Q1. List All Distinct Users and Their Stats</h3> <p><strong>SQL Code:</strong></p> <pre> SELECT username, COUNT(id) as total_submission, SUM(points) as points_earned FROM user_submissions GROUP BY 1; </pre> <p><strong>Explanation:</strong> This query aggregates the data by <code>username</code> to provide a high-level summary of engagement, showing exactly how many times each user interacted with the platform and their cumulative score.</p>

<h3>Q2. Calculate the Daily Average Points for Each User</h3> <p><strong>SQL Code:</strong></p> <pre> SELECT TO_CHAR(submitted_at, 'DD-MM') AS day, username, AVG(points) AS daily_average_points FROM user_submissions GROUP BY 1, 2; </pre> <p><strong>Explanation:</strong> This breaks down performance by date. By converting the timestamp to a 'DD-MM' format, we can see the average quality of a user's submissions for every specific day they were active.</p>

<h3>Q3. Find the Top 3 Users with the Most Correct Submissions for Each Day</h3> <p><strong>SQL Code:</strong></p>

<pre> WITH daily_submission AS ( SELECT TO_CHAR(submitted_at, 'DD-MM') AS daily, username, SUM(CASE WHEN points > 0 THEN 1 ELSE 0 END) AS total_correct_submissions FROM user_submissions GROUP BY 1, 2 ), users_rank AS ( SELECT daily, username, total_correct_submissions, DENSE_RANK() OVER (PARTITION BY daily ORDER BY total_correct_submissions DESC) AS submission_rank FROM daily_submission ) SELECT * FROM users_rank WHERE submission_rank <= 3; </pre>

<p><strong>Explanation:</strong> This uses a CTE to first count "correct" submissions (points > 0). It then applies <code>DENSE_RANK()</code> to rank users daily, allowing us to filter for the top 3 performers per day even if multiple users have the same score.</p>

<h3>Q4. Find the Top 5 Users with the Highest Number of Incorrect Submissions</h3> <p><strong>SQL Code:</strong></p> <pre> SELECT username, SUM(CASE WHEN points < 0 THEN 1 ELSE 0 END) AS total_incorrect_submissions FROM user_submissions GROUP BY 1 ORDER BY total_incorrect_submissions DESC LIMIT 5; </pre> <p><strong>Explanation:</strong> This identifies users struggling with accuracy. It uses a <code>CASE</code> statement to count only those instances where points are negative, helping administrators identify which users might need more guidance.</p>

<h3>Q5. Find the Top 10 Performers for Each Week</h3> <p><strong>SQL Code:</strong></p> <pre> SELECT * FROM ( SELECT EXTRACT(WEEK FROM submitted_at) AS week_no, username, SUM(points) AS total_points, DENSE_RANK() OVER(PARTITION BY EXTRACT(WEEK FROM submitted_at) ORDER BY SUM(points) DESC) AS rankings FROM user_submissions GROUP BY 1, 2 ) as subquery WHERE rankings <= 10; </pre> <p><strong>Explanation:</strong> This scales the analysis to a weekly view. It extracts the week number from the submission date and ranks the top 10 point-earners for that specific week, useful for periodic rewards or recognition.</p>

<h2>3. Project Conclusion</h2> <p>This EDA provides a framework for tracking user progress and platform health. By automating these queries, we can generate real-time leaderboards, identify churn risks through declining activity, and recognize high-performing users.</p>
