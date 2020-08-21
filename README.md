# ZeroDowntime
This repo contains the slides and demo scripts for my Zero Downtime database deploys talk.

# Purpose of demo
You are trying to build a new feature, but you are concerned about performance. You decide that it would be beneficial
to deploy the code to production, but to hide it behind a Feature Toggle / Feature Flag. This will allow you to test
the performance of your code, on production workloads, before you expose it to your users. It will also make it really
easy to enable, disable or throttle the new code based on performance so you can easily roll it back or throttle it 
down avoid crashing the server while you try to improve performance.

If all goes well, you should be able to monitor your improvements based on rich feedback from production. By the time 
you are ready to enable the feature in production, and expose it to your users, you should be highly confident that
it will perform as required. What's more, if it doesn't work, you should be able to disable it with ease, effectively
rolling back the change with minimul effort or risk.

# Demo instructions
1. Create the demo databases by running CREATE_DATABASES.sql against a demo SQL instance.
2. Insert the demo data by running INSERT_DATA.sql.

    You should now have a Facebook and a Toggles database.
    The Toggles database should have a Feature table containing a single row of data (a feature called 'live buttons')
    The Facebook database should have two tables, Relationship and User. Each should be populated with sample data.
    The Facebook database should have a stored procedure called FriendsOf, which can return either an old (fast but 
        basic) or a new (slow but detailed) versions of a report on a given users friends.
    If you would like to experiment with more/less test data you can user the Redgate SQL Data Generator file to experiment
        with different sized datasets. (SQL Data Generator comes with a free trial licence, but if you've used it you'll
        need a licence.)
   
3. Execute the following command against the Facebook database:

    EXEC dbo.FriendsOf @UserId = 5
    
    This should return all the IDs of the given Users friends. It's fast but basic.
    
4. Update the data in Toggles.dbo.Feature such that the Feature 'live buttons' has 'darkLanuch' enabled and rerun the 
       stored procedure. This should run more slowly but return the same data. It's running the new code in the backgroud
       but only showing the original code.
5. This performance won't do. If all the users are using the expensive version of the query you'll crash the server. You 
       need to throttle down your dark launch which you tune the query. Go back to the Toggles.dbo.Feature table and set
       the Throttle to 50. Now only 50% of executions will run the new, poor performing code. Try running the sproc again 
       and note that 50% of the time it runs fast and 50% of the time it runs slowly. You can also check the messages to
       confirm which version of the code was executed.
6. Now tune the query. Observe that the the FriendsOf stored procedure uses dynamic SQL to switch between the new and the
       old version of the code. Roughly halfway throught the script is the @NewSql script which contains a dreaded cursor.
       Even worse, this Cursor includes the line:
              WAITFOR DELAY''00:00:00.200'';
       Either delete or comment out this line.
7. Try re-running the sproc. This time it should run faster. Throttle back up to 100. Now the code should run nice and 
       fast every time, while running the new code in the background.
8. Enable the feature by going back to Toggles.dbo.Feature and flip Enabled from a 0 to a 1 for the 'live buttons feature.
9. Re-run the procedure one more time. Now it should not only run quickly, but it should also return a much richer dataset.
