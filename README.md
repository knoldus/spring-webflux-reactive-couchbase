# spring-webflux-reactive-couchbase
This is a Spring Web Flux application which interacts with Couchbase using Reactive APIs.

We have covered different flavours of querying the Couchbase using Spring Web Flux Reactive Couchbase API.

### Using auto-generated methods:
```
public interface UserRepository extends ReactiveCouchbaseRepository<User, String> {
    Mono<User> findByFirstNameAndCity(String firstName, String city);
}
```
As the repository interface extends the ReactiveCouchbaseRepository, we get the autogenerated methods to query the document on the basis of the defined model's fields.

As you can see in UserRepository, findByFirstNameAndCity is an auto-generated method which will return a single document on the basis of FirstName and City. You will get more such methods.

### Using @Query annotation:
```
@Repository
public interface ClubMemberProfileRepository extends ReactiveCouchbaseRepository<ClubMemberProfile, String> {

    @Query("select META().id AS _ID, META().cas AS _CAS, * from #{#n1ql.bucket} where clubCode = $1 and joiningDate = $2")
    Flux<ClubMemberProfile> getClubMembers(String clubCode, String joiningDate);

    @Query("select clubCode from #{#n1ql.bucket} where location = $1 and brand = $2")
    Flux<String> getAllClubCodes(String location, String brand);

}
```
Sometimes, we get the use case where the auto-generated method doesn't fit. In these scenarios, we can use @Query annotation where we can provide our custom query and can get query the document.

You would have a question here that in getClubMembers(), what is the use of _ID and _CAS. So the answer is: If we do not add this _ID and _CAS, we get the following exception:
```
Unable to retrieve enough metadata for N1QL to entity mapping, have you selected _ID and _CAS?; nested exception is rx.exceptions.OnErrorThrowable$OnNextValue: OnError while emitting onNext value: com.couchbase.client.java.query.DefaultAsyncN1qlQueryRow.class
```

### Using Joins
This is not a different way of a query actually, I just want to show here that how we can apply joins in couchbase.

This is not different than SQL. It's just we need to provide the join query in @Query annottaion. That's it.
```
public interface ClubDataRepository extends ReactiveCouchbaseRepository<ClubData, String> {

    String query = "select cmp.clubCode, cmp.joiningDate, cmp.userId, u.firstName, u.lastName," +
            " META(cmp).id AS _ID, META(cmp).cas AS _CAS from #{#n1ql.bucket} cmp" +
            " INNER JOIN #{#n1ql.bucket} u on KEYS cmp.userId" +
            " where cmp.clubCode = $1 and cmp.joiningDate = $2";

    @Query(query)
    Flux<ClubData> getClubData(String clubCode, String joiningDate);
}
```
The above query is the example of joins in couchbase. I have applied Join on u (User) and cmp (ClubMemberProfile) document on the basis of document id of User and userId of ClubMemberProfile where clubCode and joinindDate should be matched with the given values.

There are a few things to notice that:

* I have used #{#n1ql.bucket} here which actually will read the bucket name from the couchbase configuration. You do not need to provide it explicitly.
* I have used META(cmp).id AS _ID, META(cmp).cas AS _CAS here. It is for the same reason which we have already discussed above (to avoid the exception)
* To get the variable's value, I have used $1 and $2. $1 is for clubCode and $2 is for joiningDate.
