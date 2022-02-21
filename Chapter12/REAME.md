### Building and running automated tests

So now we build and run verification tests of the system landscape, as follows:

First, build the Docker images with the following commands:
```
cd $BOOK_HOME/Chapter12
./gradlew build && docker-compose build
```
Next, start the system landscape in Docker and run the usual tests with the following command:
```
./test-em-all.bash start
```