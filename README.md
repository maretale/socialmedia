<h1>Group G02 - Final Project</h1>
Nicolas Cloutier (nac1, SEAS), Crystal Lu (cryslu, SEAS), Alexander Maret (maret, SAS), Zheng Xing (xingz126, SEAS).

<h2>Features implemented</h2>
We implemented all features described in the project specifications. Specifically:

- User registration, including checking for properly formatted emails, dates, and 2 interests declared.
- Account changes, including changing affiliation, interests, and profile picture (which trigger posts to a user's page as well as all of their friends' pages), as well as email, password, and account visibility changes, which do not. Also includes account deletion.
- Walls, including friendships, posts, status updates, and comments on posts, which are visible for a user on their home page or on the user page of one of their friends.
- Home pages, including a wall, a search bar to go to other users' pages, and a button to create new posts.
- Commenting, which is limited to posts that users can see.
- Search, which automatically fills in at most 5 users based on username similarity (prefixing) according to what is entered at a particular time.
- A friends page, including viewing which friends are logged in, and a friend visualizer that displays current friends as well as friends-of-friends so long as they belong to the same institution.
- Dynamic content throughout the site, which reloads walls, friends lists, group lists, group feeds, news feeds, and other information automatically on a 15 second timer.
- Chats, including a chat invite feature and group chats. Chat messages are persistent and ordered consistently, and users can join and leave chats as they please. If someone attempts to begin a new chat with the same set of members as an existing chat, they will be redirected.
- News feeds, including updates and recommendations using an absorption algorithm based on user likes, friends, and declared news interests that is integrated into the web application.
- A news search, where users can search for news articles based on keywords.
- All of these features were implemented with checks for authentication and authorization, in addition to other security concerns such as storing password on the database salted and hashed, and with a mind towards scalability.

<h2>Extra credit claimed</h2>
In addition to the above features, we implemented the following extra credit features:

**User Privacy**
Users can mark themselves public or private, and toggle this on the account page. This interfaces with the friend request feature.

**Friend Requests**
In order to befriend private accounts, users must submit friend requests which are visible on the friends page and on that user's page, and they must be requested by the private user. Public accounts can be befriended without these requests.

**Profile Pictures**
Users can upload profile pictures, visible on their account settings page as well as their user page by other users.

**Image Content**
Users can post images along with their posts.

**Groups (including group privacy)**
Users can start groups, which function as insulated group post feeds. A group can be public or private, with private groups requiring a request to join (which must be approved by the group creator, who cannot leave the group), and public groups being joinable by anyone without requesting. Once a user joins a group, they can see posts and comments made on that group, which function similarly to user feeds above.

<h2>Source files</h2>

**Root**
- `app.js`
- `loader.js`

**Models**
- `models/chats.js`
- `models/news.js`
- `models/posts.js`
- `models/users.js`

**Routes**
- `routes/chats.js`
- `routes/news.js`
- `routes/posts.js`
- `routes/users.js`

**Views**
- `views/partials/sidebar.ejs`
- `views/account.ejs`
- `views/chat.ejs`
- `views/chatinvites.ejs`
- `views/chats.ejs`
- `views/createchat.ejs`
- `views/createcomment.ejs`
- `views/creategroup.ejs`
- `views/creategroupcomment.ejs`
- `views/creategrouppost.ejs`
- `views/createpost.ejs`
- `views/friends.ejs`
- `views/group.ejs`
- `views/grouprequests.ejs`
- `views/groups.ejs`
- `views/home.ejs`
- `views/main.ejs`
- `views/news.ejs`
- `views/signup.ejs`
- `views/user.ejs`

**News Services**
- `news/newsFeedService.js`
- `news/newsLoader.js`
- `news/newsSearch.js`
- `news/sparkIntegration.js`
- `news/Config.java`
- `news/DynamoConnector.java`
- `news/LoadNetwork.java`
- `news/SparkConnector.java`

**Spark/Livy Jobs**
- `src/main/java/news/livy/ComputeNewsRecommendations.java`
- `src/main/java/news/livy/Config.java`
- `src/main/java/news/livy/GraphBuilder.java`
- `src/main/java/news/livy/MyPair.java`
- `src/main/java/news/livy/NewsRecommendationJob.java`
- `src/main/java/news/livy/SparkConnector.java`

**Public Assets**
- `public/css/sidebar.css`
- `public/friend_visualizer/base.css`
- `public/friend_visualizer/friendvisualizer.js`
- `public/friend_visualizer/jit.js`

**Friend Visualizer**
- `friend_visualizer/base.css`
- `friend_visualizer/friendvisualizer.js`
- `friend_visualizer/jit.js`

**Utilities**
- `utils/fileUpload`

<h2>Declaration</h2>
We declare that all of the code submitted with this assignment was written by us and not copied from the internet or other people.

<h2>Instructions to run code</h2>

### Basic Setup
Run the following commands (note that the news loader loads the large news dataset, and it will take some time to run):
```
npm install
node loader.js
node news/newsLoader.js
node app.js
```
The `loader.js` script creates all required DynamoDB tables, and `newsLoader.js` populates the news-related tables.

**Important:** For the news recommendation feature to work properly, you must also set up the Spark/Livy job. See the next section for details.

### Running the News Recommendation Spark Job (Livy)

The news recommendation system uses Apache Spark via Livy to compute personalized article weights using an adsorption algorithm.

**Prerequisites:**
- Java 8+ installed
- Maven installed
- An AWS EMR cluster with Livy enabled
- AWS credentials configured (for DynamoDB access)

**Steps to run the Spark job:**

1. **Build the JAR file:**
   ```
   mvn clean package
   ```
   This creates `target/news-recommendation.jar` which contains the Livy client and job code.

2. **Start an EMR cluster with Livy:**
   - Create a new EMR cluster on AWS with Spark and Livy applications enabled
   - Note the master node public DNS (e.g., `ec2-xx-xx-xx-xx.compute-1.amazonaws.com`)

3. **Set the LIVY_URL environment variable:**
   ```
   export LIVY_URL=http://<your-emr-master-dns>:8998/
   ```

5. **Automatic execution:**
   When you run `node app.js`, the Spark job is automatically triggered:
   - Runs immediately on startup
   - Runs hourly thereafter
   - Also triggers when a user changes their interests

**Notes:**
- The Spark job reads from DynamoDB tables: `users`, `interests`, `friends`, `article_likes`, and `articles`
- Results are stored in the `article_weights` DynamoDB table