<h1>User Submissions Performance Analysis Project</h1>

<p>This project analyzes user engagement and performance data from a submission-based platform. Using PostgreSQL, we process submission patterns, point distributions, and competitive rankings. The analysis utilizes <strong>Common Table Expressions (CTEs)</strong>, <strong>Window Functions</strong>, and <strong>Conditional Aggregations</strong> to transform raw logs into actionable performance metrics.</p>

<h2>1. Database Schema and Infrastructure</h2> <p>The analysis is built upon a structured table that records every user interaction. We utilize <code>SERIAL</code> for primary keys and <code>TIMESTAMP WITH TIME ZONE</code> to maintain temporal accuracy for global users.</p>

<p><strong>Database Initialization Code:</strong></p> <pre> -- Step 1: Create the table structure CREATE TABLE user_submissions ( id SERIAL PRIMARY KEY, user_id BIGINT, question_id INT, points INT, submitted_at TIMESTAMP WITH TIME ZONE, username VARCHAR(50) );

-- Step 2: Verify that the data has been imported correctly SELECT * FROM user_submissions; </pre>

<h2>2. Exploratory Data Analysis (EDA)</h2>

<p>The following questions address user activity, accuracy, and competitive rankings.</p>

<h3>Q1. List All Distinct Users and Their Stats</h3>

<p><strong>SQL Code:</strong></p> <pre> SELECT username, COUNT(id) as total_submission, SUM(points) as points_earned FROM user_submissions GROUP BY 1; </pre> <p><strong>Explanation:</strong> This query aggregates the data by <code>username</code> to provide a high-level summary of engagement. It calculates the total number of submissions made by each user and their cumulative points earned to date.</p>

<h3>Q2. Calculate the Daily Average Points for Each User</h3>

<p><strong>SQL Code:</strong></p> <pre> SELECT TO_CHAR(submitted_at, 'DD-MM') AS day, username, AVG(points) AS daily_average_points FROM user_submissions GROUP BY 1, 2; </pre> <p><strong>Explanation:</strong> By converting the timestamp to a 'DD-MM' string, we can track consistency. This query reveals the average quality (points) of a user's output for every specific calendar day they participated.</p>

<h3>Q3. Find the Top 3 Users with the Most Correct Submissions for Each Day</h3>

<p><strong>SQL Code:</strong></p>

<pre> WITH daily_submission AS ( -- Subquery to filter and count only positive point submissions SELECT TO_CHAR(submitted_at, 'DD-MM') AS daily, username, SUM(CASE WHEN points > 0 THEN 1 ELSE 0 END) AS total_correct_submissions FROM user_submissions GROUP BY 1, 2 ), users_rank AS ( -- Window function to rank users by performance per day SELECT daily, username, total_correct_submissions, DENSE_RANK() OVER (PARTITION BY daily ORDER BY total_correct_submissions DESC) AS submission_rank FROM daily_submission ) -- Filter to show only the podium finishers (Top 3) SELECT * FROM users_rank WHERE submission_rank <= 3; </pre>

<p><strong>Explanation:</strong> This uses a CTE to isolate "correct" submissions (those where points > 0). It then applies <code>DENSE_RANK()</code> to create a daily leaderboard. Unlike a standard rank, <code>DENSE_RANK</code> ensures that if two people tie for 1st, the next person is ranked 2nd rather than 3rd.</p>

<h3>Q4. Find the Top 5 Users with the Highest Number of Incorrect Submissions</h3>

<p><strong>SQL Code:</strong></p> <pre> SELECT username, SUM(CASE WHEN points < 0 THEN 1 ELSE 0 END) AS total_incorrect_submissions FROM user_submissions GROUP BY 1 ORDER BY total_incorrect_submissions DESC LIMIT 5; </pre> <p><strong>Explanation:</strong> This serves as a quality control metric. By specifically counting submissions where <code>points < 0</code>, we identify users who may be guessing or struggling with specific difficulty levels, allowing for better platform moderation.</p>

<h3>Q5. Find the Top 10 Performers for Each Week</h3>

<p><strong>SQL Code:</strong></p> <pre> SELECT * FROM ( SELECT EXTRACT(WEEK FROM submitted_at) AS week_no, username, SUM(points) AS total_points, DENSE_RANK() OVER(PARTITION BY EXTRACT(WEEK FROM submitted_at) ORDER BY SUM(points) DESC) AS rankings FROM user_submissions GROUP BY 1, 2 ) as weekly_summary WHERE rankings <= 10; </pre> <p><strong>Explanation:</strong> This expands the scope to a weekly timeframe. It extracts the ISO week number from the date and identifies the top 10 point-earners for that week, which is ideal for periodic "hall of fame" reports or weekly prize distributions.</p>

<h2>3. Project Conclusion</h2> <p>This EDA provides a powerful framework for tracking platform engagement. By segregating the logic into daily and weekly tiers, we can monitor long-term user growth while also pinpointing immediate issues with submission accuracy.</p>
