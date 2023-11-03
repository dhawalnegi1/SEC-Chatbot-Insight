# Assignment3

### Project Descrition 

This project builds upon the foundation laid by Project 1(https://github.com/BigDataIA-Fall2023-Team2/Assignment1/blob/main/part1/Readme.md) and Project 2(https://github.com/BigDataIA-Fall2023-Team2/Assignment2). Here, we are using forms uploaded in the sec website (https://www.sec.gov/forms). The application developed in the project is a chatbot that can answer any question related to any of the uploaded forms in the sec website (https://www.sec.gov/forms) and answer the question raised by user. There are three parts to the project. 

1) Part 1 - We are creating two airflows. First airflow is to fetch the contents of the pdfs from the sec website and create emmbeddings for the same. Second airflow is to then store these embeddings in the pinecone vector database.

2) Part 2 - Once the user asks any question, the embeddings for the question will be created and the closest embeddings will be fetched from the vector database. The contents of that particular embedding will be then passed to OpenAI to answer the query of the user.

3) Part 3 - The user will be prompted to login to the application first. If the user is not regsitered, a registration page will prompt user to register after which the user has to logsin. Once logged in, a JWT has been created for the user. If the token becomes invalid, then the user has to relogin to use the application again.  

### Application and Documentation Link

App link - 

Fast API hosted on Google VM - 

### Project Resources

Google Codelab link - 

Google Collab Notebook link -

Project Demo - 

### Tech Stack
Python | Streamlit | Google Collab | Nougat | Google VM | Fast API | Airflow | PyPDF | Docker | MS-SQL sever

### Architecture diagram ###

![image](https://github.com/BigDataIA-Fall2023-Team2/Assignment3/assets/131703516/2915a047-cabf-4092-9c44-d839314491ce)

### Project Flow

The app is a chatbot application that will answer questions related to the forms uploaded in SEC website. 

1) Part 1 - The contents of the pdfs present in the sec website is fetched and chunks of the content is created. Then for each and every chunk, we are storing the metadata of the chunk such as pdf name, pdf number, number of tokens etc. We are creating emmbeddings for each and every chunk. All this information will be stored in a csv file. All of this is being done inside an aiflow. This airflow is then deployed to google VM for execution. 

2)  Part 2 - The embeddings and the related metadata fetched in the csv file which is getting generated from airflow in part1 will be stored in a pinecone vector database and an SQL server database. The idea is to store only the embeddings in the pinecone database and its respective content from PDF (that will be used to pass to OPENAI) will be stored in SQL server database. All of this is being done inside an another aiflow. This airflow is then deployed to google VM for execution. 

3)  Part 3 - The user will login/register into the application. Credentials of the user will be stored in a sql server (with the password being stored as a hash value). These interaction of streamlit app with SQL server will happen through FAST API that too via secure endpoints. After the user succesfully logs in, a JWT will be generated. This will in turn keep a track of the user's session. Once the token becomes invalid, the user will be prompted to re-login to use the application. The user needs to then select the pdf related to the questions/doubts he has. After this, the user will ask the question in the application. The streamlit app will then create an embeddings for the question and pass it to the pinecone database. The pinecone database will be used to find the context that can be the closest to the question based on the closest embeddings. Once the closest embeddings is found, it will pass the id of this embedding to sql server. The corresponding content/chunk  of the id contains the answer to the question. This chunk is then passed to OpenAI, and the answer to the question is sent back to the application. 

All the microservices has been containerized and hosted on cloud/publicly available platforms.

### Repository Structure



### Contributions

| Name                            | Contribution                               |  
| ------------------------------- | ---------------------------------------------------------------------|
| Shardul Chavan                  | Part 1 - Airflow, Embeddings                                         | 
| Chinmay Gandi                   | Part 2 - Aiflow, Pinecone DB, MS-SQL                                 |
| Chinmay Gandi, Dhawal Negi      | User login/registration, JWT, FAST API                               | 
| Dhawal Negi                     | Airflow deployment to google VM, Integration of part1, part2, part3  |              

### Additional Notes
WE ATTEST THAT WE HAVEN’T USED ANY OTHER STUDENTS’ WORK IN OUR ASSIGNMENT AND ABIDE BY THE POLICIES LISTED IN THE STUDENT HANDBOOK. 

