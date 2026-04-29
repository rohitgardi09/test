Entity
➡️ Represents the database table structure used to store exception log records.
DTO (Data Transfer Object)
➡️ Carries exception log data between layers without exposing the entity directly.
DAO (Data Access Object)
➡️ Handles custom database operations and acts as an intermediate layer between service and repository.
Repository
➡️ Provides built-in CRUD operations for the exception log entity using Spring Data JPA.
Mapper
➡️ Converts data between DTO and Entity objects for persistence and retrieval.
Service
➡️ Captures exceptions, enriches them with context (stacktrace, path, correlationId), and stores them in the database.
