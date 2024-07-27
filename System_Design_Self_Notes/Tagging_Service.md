**Atlassian Interview Question | System Design: Tagging Service**

https://www.youtube.com/watch?v=WNIR7eiv0Hk&list=PLlvnxKilk3aIKa3Vv688ELv-DeAEg3YQ1&index=1

**Tagging Service Design Article**

https://levelup.gitconnected.com/tagging-service-system-design-ee0081aa0086

# Requirements

* Tag an item
* View the items with a specific tag in near real-time
* Scalable

# Data storage

## **Database schema**

![](https://miro.medium.com/v2/resize:fit:1014/1*4VHjoZWObuujtt9sLoSMYw.png)

Tagging service; Database schema

* The primary entities of the database are the Tags table, the Items table, and the Tags_Items table
* The Tags_Items is a join table to represent the relationship between the Items and the Tags
* The relationship between the Tags and the Items tables is many-to-many

## **Type of data store**

* The media files (images, videos) and text files are stored in a managed [object storage](https://en.wikipedia.org/wiki/Object_storage) such as AWS [S3](https://aws.amazon.com/s3/)
* A SQL database such as Postgres stores the metadata on the relationship between tags and items
* A NoSQL data store such as MongoDB stores the metadata of the item
* A cache server such as Redis stores the popular tags and items

# High-level design

1. When a new item is tagged, the metadata is stored on the SQL database
2. The popular tags and items are cached on dedicated cache servers to improve latency
3. The non-popular tags and items are fetched by querying the read replicas of SQL and NoSQL data stores

# Write path

![](https://miro.medium.com/v2/resize:fit:1050/1*K-UzMuwhYLqzNzwZYDG63w.png)

**Tagging service; Write path**

1. The client makes an HTTP connection to the load balancer
2. The load balancer delegates the client request to a web server with free capacity
3. The write requests to create an item or tag an item are rate limited
4. The write requests are stored on the message queue for asynchronous processing and improved fault tolerance
5. The fanout service distributes the write request to multiple services to tag an item
6. The object store persists the text files or media files embedded in an item
7. The NoSQL data store persists the metadata of an item (comments, upvotes, published date)
8. The SQL database persists metadata on the relationship between tags and items
9. The tags info service is queried to identify the popular tags
10. If the item was tagged with a popular tag, the item is stored on the items cache server
11. The tags cache server stores the IDs of items that were tagged with popular tags
12. LRU cache is used to evict the cache servers
13. The data objects (items and tags) are replicated across data centers at the web server level to save bandwidth

# Read path

![](https://miro.medium.com/v2/resize:fit:1050/1*CxWYI2IhEY7VDRe-_lRLYA.png)

**Tagging service; Read path**

1. The client executes a DNS query to resolve the domain name
2. The client queries the CDN to check if the tag data is cached on the CDN
3. The client creates an HTTP connection to the load balancer
4. The load balancer delegates the client request to a web server with free capacity
5. The read requests to fetch the tags or items are rate limited
6. The web server queries the tags service to fetch the tags
7. The tags service queries the tags info service to identify if the requested tag is popular
8. The lists of tagged items for a popular tag are fetched from the tags cache server
9. The tags service executes an MGET Redis request to fetch the relevant tagged items from the items cache server
10. The list of items tagged with non-popular tags is fetched from the read replicas of the SQL database
11. The items tagged with non-popular tags are fetched from the read replicas of the NoSQL data store
12. The media files embedded in an item are fetched from the object store
13. The trie data structure is used for typeahead autosuggestion for search queries on tags

# How to Design a Database For Tagging Service?

https://www.geeksforgeeks.org/how-to-design-a-database-for-tagging-service/

Tagging services are important for ****organizing**** and ****sorting ****different kinds of content like  ****articles**** , images, products, and documents. They let users add descriptive tags or keywords to items, making it easy to search for and find them.

Designing a good database for a tagging service means thinking about how the data is  ****structured**** , how well the system can  ****grow**** , how to keep it running quickly, and how to make it user-friendly. In this article, we’ll look at the key ideas for creating databases that are perfect for tagging services.

## Database Design for Tagging Services

Designing a database for a ****tagging service**** requires careful thought about several important factors to make sure tagged items are ****organized ****and ****retrieved ****efficiently. A well-structured database is essential for managing tags,  ****tagged items**** ,  ****user interactions**** , and the relationships between them, ensuring a smooth tagging experience.

## Features of Databases for Tagging Services

Databases for tagging services offer a range of features designed to enhance user experience and optimize platform performance. These features typically include:

* ****Tag Management:**** Managing tag creation, editing, and deletion.
* ****Item Tagging:**** Allowing users to tag items with descriptive keywords or phrases.
* ****Search and Filtering:**** Enabling users to search for tagged items based on specific tags or filter criteria.
* ****Tag Recommendations:**** Providing users with recommendations for relevant tags based on item content or user behavior.
* ****User Profile and Preferences:**** Managing user accounts, preferences, and interactions with tagged items.
* **Tagging Analytics**: Generating insights and analytics to track tag usage, popular tags, and user engagement.

## Entities and Attributes in Databases for Tagging Services

[Entities ](https://www.geeksforgeeks.org/difference-between-entity-entity-set-and-entity-type/)in a tagging service database represent different parts of tagged items, tags, user interactions, and the connections between them, while [attributes ](https://www.geeksforgeeks.org/types-of-attributes-in-er-model/)describe their features. Common entities and their attributes include

### Item Table

* **ItemID (Primary Key**): Unique identifier for each tagged item.
* ****Title, Description:**** Metadata for item title and description.
* ****ItemType:**** Type of the tagged item (e.g., article, image, product).
* ****Content:**** Content of the tagged item (e.g., text, image URL, product details).

### Tag Table

* ****TagID (Primary Key):**** Unique identifier for each tag.
* ****TagName:**** Name or keyword associated with the tag.

### User Table

* ****UserID (Primary Key):**** Unique identifier for each user.
* ****Username, Email:**** User’s username and email address.
* ****PasswordHash:**** Securely hashed password for user authentication.

### Tagging Table

* ****TaggingID (Primary Key):**** Unique identifier for each tagging interaction.
* ****UserID:**** Identifier for the user who tagged the item.
* ****ItemID:**** Identifier for the tagged item.
* ****TagID:**** Identifier for the tag associated with the item.
* ****Timestamp:**** Date and time of the tagging interaction.

## Relationships Between Entities

Based on the entities and their attributes provided, let’s define the relationships between them

### Many-to-Many Relationship between Item and Tag

* One item can be associated with multiple tags.
* One tag can be associated with multiple items.
* Therefore, the relationship between Item and Tag is [many-to-many](https://www.geeksforgeeks.org/relationships-in-sql-one-to-one-one-to-many-many-to-many/).

### Many-to-Many Relationship between User and Tagging

* One user can tag multiple items.
* One item can be tagged by multiple users.
* Therefore, the relationship between User and Tagging is many-to-many.

## Entity Structures in SQL Format

Here’s how the entities mentioned above can be structured in [SQL ](https://www.geeksforgeeks.org/sql-tutorial/)format

```
-- Table: Item
CREATE TABLE Item (
    ItemID INT PRIMARY KEY AUTO_INCREMENT,
    Title VARCHAR(255) NOT NULL,
    Description TEXT,
    ItemType VARCHAR(50),
    Content TEXT
);

-- Table: Tag
CREATE TABLE Tag (
    TagID INT PRIMARY KEY AUTO_INCREMENT,
    TagName VARCHAR(100) NOT NULL
);

-- Table: User
CREATE TABLE User (
    UserID INT PRIMARY KEY AUTO_INCREMENT,
    Username VARCHAR(50) NOT NULL,
    Email VARCHAR(100) NOT NULL,
    PasswordHash VARCHAR(255) NOT NULL
);

-- Table: Tagging
CREATE TABLE Tagging (
    TaggingID INT PRIMARY KEY AUTO_INCREMENT,
    UserID INT NOT NULL,
    ItemID INT NOT NULL,
    TagID INT NOT NULL,
    Timestamp DATETIME NOT NULL,
    FOREIGN KEY (UserID) REFERENCES User(UserID),
    FOREIGN KEY (ItemID) REFERENCES Item(ItemID),
    FOREIGN KEY (TagID) REFERENCES Tag(TagID)
);
```

## Database Model for Tagging Services

The database model for tagging services revolves around efficiently managing tagged items, tags, user interactions, and relationships between them to provide a seamless tagging experience.

![DB_Design_Tagging](https://media.geeksforgeeks.org/wp-content/uploads/20240514122714/DB_Design_Tagging.webp)DB_Design_Tagging

## Tips & Best Practices for Enhanced Database Design

* ****Indexing:**** Implement indexing on frequently queried columns to improve search and retrieval performance.
* ****Normalization:**** Normalize data to reduce [redundancy ](https://www.geeksforgeeks.org/the-problem-of-redundancy-in-database/)and improve database efficiency.
* ****Asynchronous Processing:**** Implement asynchronous processing for tag recommendations and analytics to minimize impact on user experience.
* ****Data Validation:**** Implement data validation mechanisms to ensure consistency and integrity of tagged items and tags.
* ****Scalability:**** Design the database with [scalability](https://www.geeksforgeeks.org/what-is-scalability-and-how-to-achieve-it-learn-system-design/) in mind to accommodate future growth in user base and tagged content.


# Database Design for Tagging Service — Learn how the load on your system can impact your design.

https://medium.com/technical-insights/database-design-for-tagging-service-3-approaches-b263b2a187b9

# Requirements for Tagging Service:

1. Add a tag to an item (Jira ticket, Confluence page, StackOverflow question etc)
2. Search for a tag and get all items that have that tag

# Use cases

We will consider 3 use cases and will see the approach for each use case.

1. Low read and low writes
2. Heavy reads and low writes (read heavy)
3. Heavy reads and heavy writes (read-write heavy)

# Use case 1 (low read and low writes)

1. Use SQL db as data is relational.
2. Tags and items will have many-to-many mapping.
3. Schema and Queries

```
-- Table for posts
CREATE TABLE posts (
    post_id SERIAL PRIMARY KEY,
    post_content TEXT,
    tags TEXT  -- this is optional
);

-- Table for tags
CREATE TABLE tags (
    tag_id SERIAL PRIMARY KEY,
    tag_name VARCHAR(255)
);

-- Junction table to associate posts with tags
CREATE TABLE post_tags (
    post_id INT,
    tag_id INT,
    FOREIGN KEY (post_id) REFERENCES posts(post_id),
    FOREIGN KEY (tag_id) REFERENCES tags(tag_id),
    PRIMARY KEY (post_id, tag_id)
);

-- Data insertion
insert into post (post_content, tags)
values
('post_1', '#t1 #t2 #t3'),
('post_2', '#t1 #t4 #t5'),
('post_3', '#t2 #t4 #t5');

insert into tags (tag_name)
values
('t1'),
('t2'),
('t3'),
('t4'),
('t5');

insert into post_tags (post_id, tag_id)
values
(1,1),
(1,2),
(1,3),
(2,1),
(2,4),
(2,5),
(3,2),
(3,4),
(3,5);


-- Query to get list of posts based on a particular tag
SELECT p.post_id, p.post_content
FROM posts p
JOIN post_tags pt ON p.post_id = pt.post_id
JOIN tags t ON pt.tag_id = t.tag_id
WHERE t.tag_name = 't1';
```

# Use case 2 (heavy reads and low writes)

Similar to use case 1. Have multiple read replicas of your RDBMS to handle heavy read load.

# Use case 3 (read-write heavy)

1. SQL won’t work here as the system is write-heavy. So we will use NoSQL DB. Cassandra seems to be a good option here based on the data and query pattern.
2. The partitioning key can be `tag_id` so that all data related to a tag is stored in a single partition.
3. Schema and Queries

```
CREATE TABLE posts (
    post_id UUID PRIMARY KEY,
    title TEXT,
    content TEXT,
    tags SET<TEXT>
);

CREATE TABLE tags_by_post (
    post_id UUID,
    tag TEXT,
    PRIMARY KEY (tag, post_id)
);

-- Query TagsByPost table to get post IDs for a specific tag
SELECT post_id FROM tags_by_post WHERE tag = 'your_tag';

-- Fetch posts based on post IDs retrieved
SELECT * FROM posts WHERE post_id IN (post_id1, post_id2, ...);
```
