# lab 8 todo

only the exercise counts (p.20-21)

## répartition

- Clark : `EmployeesResolver.kt` (branche `clark/resolver`)
- Mathys : `employee.graphqls` + le rapport
- noms à garder identiques : `employees`, `newEmployee`, `createEmployeeInput`

## what we have to do

- `server/` already has the Employee entity, EmployeesRepository, CreateEmployeeInput
- `employee.graphqls` is empty and `EmployeesResolver` is an empty class, that's what we fill in
- the client only sends 2 things (in `client/src/app/service/employee.service.ts`)
  - the `employees` query, gets the list of employees (id, name, dateOfBirth, city, salary, gender, email)
  - the `newEmployee` mutation, adds an employee and gets it back
- so the schema needs `employees` under `type Query` and `newEmployee` under `type Mutation`, plus the `Employee` type and the `CreateEmployeeInput` input they use

## running it

- mongo (from PDF, p.2)

```
docker run -d -p 27017:27017 --name mongodb -d mongo
```

- server with `gradlew.bat bootRun` in `server/`
- client with `npm install` then `npm start` in `client/`

## employee.graphqls

- `Employee` fields are the same as `entity/Employee.kt`, id is `ID!`
- `CreateEmployeeInput` fields are the same as `CreateEmployeeInput.kt`
- gender and email are nullable in Kotlin so no `!`

from PDF, p.4

```graphql
type Query {
  books: [Book]
}

type Book {
  bookId: ID!
  bookNumber: Int!
  category: String!
  title: String!
  cost: Float!
  year: String
  description: String
}

input CreateBookInput {
  bookNumber: Int
  category: String
  title: String
  cost: Float
  year: String
  description: String
}

type Mutation {
  newBook(createBookInput: CreateBookInput!): Book!
}
```

## EmployeesResolver

- add `EmployeesRepository` to the constructor
- `employees()` with @QueryMapping, returns `findAll()`
- `newEmployee(...)` with @MutationMapping and `@Argument("createEmployeeInput")`
  - check the required fields aren't null, build the Employee, set id to a UUID, save it, return it
- function names and the @Argument name have to match the schema

from PDF, p.6

```kotlin
@Controller
class BooksController(private val bookRepository: BookRepository) {
    @QueryMapping
    fun books(): List<Book> {
        return bookRepository.findAll()
    }
    ...
    @MutationMapping
    fun newBook(@Argument("createBookInput") input: CreateBookInput) : Book {
        if (input.bookNumber != null &&
            input.category != null &&
            input.title != null && input.cost != null) {
            val book =
                Book(input.bookNumber, input.category, input.title, input.cost, input.year, input.description)
            book.bookId = UUID.randomUUID().toString()
            bookRepository.save(book)
            return book
        } else {
            throw Exception("Invalid input")
        }
    }
}
```

## test

- localhost:9000/graphiql (p.8), run `employees` and `newEmployee`
- then add an employee from the client (localhost:4200)
