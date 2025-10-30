# Applying Domain-Driven Design With Enterprise Java: A Behavior-Driven Approach

When it comes to software development, one of the biggest mistakes is delivering precisely what the client wants. While this may sound cliché, the problem persists even after decades in the industry. A more effective approach is to begin testing with a focus on business needs.

[Behavior-driven development](https://dzone.com/articles/what-is-bdd-a-complete-guide) (BDD) is a software development methodology that emphasizes behavior and domain terminology, also known as ubiquitous language. It uses a shared, natural language to define and test software behaviors from the user's perspective. BDD builds on [test-driven development](https://dzone.com/articles/the-importance-of-test-driven-development-in-softw) (TDD) by concentrating on scenarios that are relevant to the business. These scenarios are written as plain-language specifications that can be automated into tests, which also serve as living documentation.

This approach promotes a common understanding between both technical and non-technical stakeholders, ensures that the software meets user needs, and helps reduce rework and development time. In this article, we will explore this methodology further and discuss how to implement it using Oracle NoSQL and Java.

## How BDD and DDD Work Together

At first glance, behavior-driven development (BDD) and domain-driven design (DDD) may appear to address different problems — one focusing on _testing_ and the other on _modeling_. Yet, both share the same philosophical foundation: ensuring that software truly reflects the **business domain** it serves.

DDD, introduced by Eric Evans in his seminal 2003 book _Domain-Driven Design: Tackling Complexity in the Heart of Software_, teaches us to model software around business concepts — entities, value objects, aggregates, and bounded contexts. Its power lies in the use of **ubiquitous language**, a shared vocabulary that unites developers and domain experts.

BDD, coined a few years later by Dan North, emerged as a natural extension of this idea. It brought ubiquitous language into the testing process, turning business rules into executable specifications. Where DDD defines _what_ the system should represent, BDD validates _how_ the system behaves according to that model.

When used together, DDD and BDD form a continuous feedback loop:

-   DDD shapes the **domain model** that captures the business logic.
-   BDD ensures that the **system behavior** stays consistent with that model over time.

In practice, this synergy means you can write feature scenarios — such as _“When I reserve a VIP room, the system should mark it as unavailable”_ — directly tied to aggregates like Room and Reservation. These tests become living documentation for both developers and stakeholders, ensuring your domain remains aligned with real business needs.

If you want to explore this alignment in depth, my book _[Domain-Driven Design with Java](https://bpbonline.com/products/domain-driven-design-with-java)_ expands on these principles. It shows how to apply DDD patterns in modern Java applications using Jakarta EE, Spring, and cloud technologies, offering a practical foundation for uniting architecture and behavior.

Together, DDD and BDD close the gap between understanding the business and proving it works — transforming software from a technical artifact into a faithful expression of the domain itself.

## Show Me the Code

In this sample, we’ll generate a simple hotel management application using Enterprise Java and the Oracle NoSQL Database.

The first step is to create the project. Since we’re working with Java SE, we can generate it using the following Maven command:

```bash
mvn archetype:generate                     \
"-DarchetypeGroupId=io.cucumber"           \
"-DarchetypeArtifactId=cucumber-archetype" \
"-DarchetypeVersion=7.30.0"                \
"-DgroupId=org.soujava.demos.hotel"        \
"-DartifactId=behavior-driven-development" \
"-Dpackage=org.soujava.demos"              \
"-Dversion=1.0.0-SNAPSHOT"                 \
"-DinteractiveMode=false"
```

The next step is to include **Eclipse JNoSQL** with **Oracle NoSQL**, along with the **Jakarta EE component implementations**: CDI, JSON, and the **Eclipse MicroProfile** implementation.

You can find the complete [pom.xml file](https://github.com/soujava/behavior-driven-development-oracle-nosql/blob/main/pom.xml).

With the initial project ready, we’ll start by creating the tests.

 Remember, BDD is an extension of TDD that includes the ubiquitous language — the shared vocabulary between the domain and the business.

```text
Feature: Manage hotel rooms

  Scenario: Register a new room
    Given the hotel management system is operational
    When I register a room with number 203
    Then the room with number 203 should appear in the room list

  Scenario: Register multiple rooms
    Given the hotel management system is operational
    When I register the following rooms:
      | number | type      | status             | cleanStatus |
      | 101    | STANDARD  | AVAILABLE          | CLEAN       |
      | 102    | SUITE     | RESERVED           | DIRTY       |
      | 103    | VIP_SUITE | UNDER_MAINTENANCE  | CLEAN       |
    Then there should be 3 rooms available in the system

  Scenario: Change room status
    Given the hotel management system is operational
    And a room with number 101 is registered as AVAILABLE
    When I mark the room 101 as OUT_OF_SERVICE
    Then the room 101 should be marked as OUT_OF_SERVICE
```

With the maven project completed, let's move to the next step, which is creating the modeling and the repository. As mentioned before, we'll focus on room management. Therefore, our next goal is to ensure that the previously defined BDD tests pass. Let's start by implementing the domain model and the repository:

```java
public enum CleanStatus {
    CLEAN,
    DIRTY,
    INSPECTION_NEEDED
}

public enum RoomStatus {
    AVAILABLE,
    RESERVED,
    UNDER_MAINTENANCE,
    OUT_OF_SERVICE
}

public enum RoomType {
    STANDARD,
    DELUXE,
    SUITE,
    VIP_SUITE
}

@Entity
public class Room {

    @Id
    private String id;

    @Column
    private int number;

    @Column
    private RoomType type;

    @Column
    private RoomStatus status;

    @Column
    private CleanStatus cleanStatus;

    @Column
    private boolean smokingAllowed;

    @Column
    private boolean underMaintenance;
  
}
```

With the model, the next step is to create the bridge between Enterprise Java and Oracle NoSQL as a non-relational database. We can do it super easily with Jakarta Data, which has a single repository, so we don't need to worry about the implementation.

```java
@Repository
public interface RoomRepository {

    @Query("FROM Room")
    List<Room> findAll();

    @Save
    Room save(Room room);

    void deleteBy();

    Optional<Room> findByNumber(Integer number);
}
```

With the project completed, the next step is to prepare the test environment, starting by making a database instance available for testing. Thanks to Testcontainers, we can easily spin up an isolated instance of Oracle NoSQL to run our tests.

```java

public enum DatabaseContainer {

    INSTANCE;

    private final GenericContainer<?> container = new GenericContainer<>
            (DockerImageName.parse("ghcr.io/oracle/nosql:latest-ce"))
            .withExposedPorts(8080);

    {
        container.start();
    }
    public DatabaseManager get(String database) {
        DatabaseManagerFactory factory = managerFactory();
        return factory.apply(database);
    }



    public DatabaseManagerFactory managerFactory() {
        var configuration = DatabaseConfiguration.getConfiguration();
        Settings settings = Settings.builder()
                .put(OracleNoSQLConfigurations.HOST, host())
                .build();
        return configuration.apply(settings);
    }

    public String host() {
        return "http://" + container.getHost() + ":" + container.getFirstMappedPort();
    }
}
```

After that, we’ll create a producer integrated with the `@Alternative` CDI annotation. This configuration teaches CDI how to provide the database instance — in this case, the one managed by Testcontainers:

```java
@ApplicationScoped
@Alternative
@Priority(Interceptor.Priority.APPLICATION)
public class ManagerSupplier implements Supplier<DatabaseManager> {

    @Produces
    @Database(DatabaseType.DOCUMENT)
    @Default
    public DatabaseManager get() {
        return DatabaseContainer.INSTANCE.get("hotel");
    }

}
```

With Cucumber, we can define an ObjectFactory that injects classes into the Cucumber test context. Since we’re using CDI with Weld as the implementation, we’ll create a custom WeldCucumberObjectFactory to integrate both technologies seamlessly.

```java
public class WeldCucumberObjectFactory implements ObjectFactory {

    private Weld weld;
    private WeldContainer container;

    @Override
    public void start() {
        weld = new Weld();
        container = weld.initialize();
    }

    @Override
    public void stop() {
        if (weld != null) {
            weld.shutdown();
        }
    }

    @Override
    public boolean addClass(Class<?> stepClass) {
        return true;
    }

    @Override
    public <T> T getInstance(Class<T> type) {
        return (T)  container.select(type).get();
    }
}
```

One important note: this setup works as an SPI (Service Provider Interface). Therefore, you must create the following file:

`src/test/resources/META-INF/services/io.cucumber.core.backend.ObjectFactory`

With the following content:

```java
org.soujava.demos.hotels.config.WeldCucumberObjectFactory
```

We will have the Mapper convert our table into the Room from all models.

```java
@ApplicationScoped
public class RoomDataTableMapper {

    @DataTableType
    public Room roomEntry(Map<String, String> entry) {
        return Room.builder()
                .number(Integer.parseInt(entry.get("number")))
                .type(RoomType.valueOf(entry.get("type")))
                .status(RoomStatus.valueOf(entry.get("status")))
                .cleanStatus(CleanStatus.valueOf(entry.get("cleanStatus")))
                .build();
    }
}
```

With the whole test infrastructure done, the next step is to design the Step tests that will contain our test itself.

```java
@ApplicationScoped
public class HotelRoomSteps {

    @Inject
    private RoomRepository repository;

    @Before
    public void cleanDatabase() {
        repository.deleteBy();
    }

    @Given("the hotel management system is operational")
    public void theHotelManagementSystemIsOperational() {
        Assertions.assertThat(repository).as("RoomRepository should be initialized").isNotNull();
    }

    @When("I register a room with number {int}")
    public void iRegisterARoomWithNumber(Integer number) {
        Room room = Room.builder()
                .number(number)
                .type(RoomType.STANDARD)
                .status(RoomStatus.AVAILABLE)
                .cleanStatus(CleanStatus.CLEAN)
                .build();
        repository.save(room);
    }

    @Then("the room with number {int} should appear in the room list")
    public void theRoomWithNumberShouldAppearInTheRoomList(Integer number) {
        List<Room> rooms = repository.findAll();
        Assertions.assertThat(rooms)
                .extracting(Room::getNumber)
                .contains(number);
    }

    @When("I register the following rooms:")
    public void iRegisterTheFollowingRooms(List<Room> rooms) {
        rooms.forEach(repository::save);
    }

    @Then("there should be {int} rooms available in the system")
    public void thereShouldBeRoomsAvailableInTheSystem(int expectedCount) {
        List<Room> rooms = repository.findAll();
        Assertions.assertThat(rooms).hasSize(expectedCount);
    }

    @Given("a room with number {int} is registered as {word}")
    public void aRoomWithNumberIsRegisteredAs(Integer number, String statusName) {
        RoomStatus status = RoomStatus.valueOf(statusName);
        Room room = Room.builder()
                .number(number)
                .type(RoomType.STANDARD)
                .status(status)
                .cleanStatus(CleanStatus.CLEAN)
                .build();
        repository.save(room);
    }

    @When("I mark the room {int} as {word}")
    public void iMarkTheRoomAs(Integer number, String newStatusName) {
        RoomStatus newStatus = RoomStatus.valueOf(newStatusName);
        Optional<Room> roomOpt = repository.findByNumber(number);

        Assertions.assertThat(roomOpt)
                .as("Room %s should exist", number)
                .isPresent();

        Room updatedRoom = roomOpt.orElseThrow();
        updatedRoom.update(newStatus);

        repository.save(updatedRoom);
    }

    @Then("the room {int} should be marked as {word}")
    public void theRoomShouldBeMarkedAs(Integer number, String expectedStatusName) {
        RoomStatus expectedStatus = RoomStatus.valueOf(expectedStatusName);
        Optional<Room> roomOpt = repository.findByNumber(number);

        Assertions.assertThat(roomOpt)
                .as("Room %s should exist", number)
                .isPresent()
                .get()
                .extracting(Room::getStatus)
                .isEqualTo(expectedStatus);
    }
}
```

Time to execute the test with:

Where you can see the results:

```bash
INFO: Connecting to Oracle NoSQL database at http://localhost:61325 using ON_PREMISES deployment type
  ✔ Given the hotel management system is operational      # org.soujava.demos.hotels.HotelRoomSteps.theHotelManagementSystemIsOperational()
  ✔ And a room with number 101 is registered as AVAILABLE # org.soujava.demos.hotels.HotelRoomSteps.aRoomWithNumberIsRegisteredAs(java.lang.Integer,java.lang.String)
  ✔ When I mark the room 101 as OUT_OF_SERVICE            # org.soujava.demos.hotels.HotelRoomSteps.iMarkTheRoomAs(java.lang.Integer,java.lang.String)
  ✔ Then the room 101 should be marked as OUT_OF_SERVICE  # org.soujava.demos.hotels.HotelRoomSteps.theRoomShouldBeMarkedAs(java.lang.Integer,java.lang.String)
Oct 21, 2025 6:18:43 PM org.jboss.weld.environment.se.WeldContainer shutdown
INFO: WELD-ENV-002001: Weld SE container fc4b3b51-fba8-4ea6-9cef-42bcee97d220 shut down
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 7.231 s -- in org.soujava.demos.hotels.RunCucumberTest
[INFO] Running org.soujava.demos.hotels.MongoDBTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.007 s -- in org.soujava.demos.hotels.MongoDBTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] 
```

## Conclusion

By combining domain-driven design (DDD) and behavior-driven development (BDD), developers can move beyond technical correctness and build software that truly mirrors business intent. DDD gives structure to the domain, ensuring that models capture real-world concepts with precision, while BDD ensures those models behave as expected through clear, testable scenarios written in the language of the business itself.

In this article, you’ve learned how to connect these two worlds using Oracle NoSQL, Eclipse JNoSQL, and Jakarta EE — from defining your domain to running real behavioral tests powered by Cucumber and CDI. This synergy transforms tests into living documentation, bridging the gap between engineers and stakeholders and ensuring your system remains aligned with business goals as it evolves.

You can go deep and combine DDD with BDD. In the [_Domain-Driven Design with Java_](https://bpbonline.com/products/domain-driven-design-with-java) book, you can find a good starting point for understanding why DDD is still important to us. It expands on the ideas shared here, showing how DDD and BDD together can lead to simpler, more maintainable, and business-focused software. This kind delivers actual value beyond requirements.
