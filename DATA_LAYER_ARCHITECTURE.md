# Data Layer Architecture Guide

This document defines the standardized approach for creating DAOs, DataSources, and Repositories in our Android project, based on analysis of the Sunshine-Photos Clean Architecture implementation.

## Core Data Architecture Principles

1. **Three-Layer Separation**: Clear separation between DAO, DataSource, and Repository layers
2. **Clean Architecture**: Domain layer defines interfaces, infrastructure implements them
3. **Reactive Programming**: Use Kotlin Flow for real-time data updates
4. **Transaction Management**: Atomic operations with proper error handling
5. **Type Safety**: Strong typing between layers with proper converters

## Project Structure

```
feature/
├── domain/                    # Business logic layer
│   ├── src/main/java/
│   │   └── com/project/feature/domain/
│   │       ├── FeatureRepository.kt        # Repository interface
│   │       ├── usecases/                  # Use cases
│   │       └── valueobjects/              # Domain value objects
│   └── build.gradle.kts
├── infrastructure/            # Implementation layer
│   ├── src/main/java/
│   │   └── com/project/feature/infrastructure/
│   │       ├── FeatureRepositoryImpl.kt   # Repository implementation
│   │       ├── FeatureDbDataSource.kt     # DataSource interface
│   │       ├── models/                    # Database DTOs
│   │       └── converters/                # Domain <-> DTO converters
│   └── build.gradle.kts
└── datasource/               # Database access layer
    ├── src/main/java/
    │   └── com/project/feature/datasource/
    │       ├── FeatureDbDataSourceImpl.kt # DataSource implementation
    │       └── daos/
    │           └── FeatureDao.kt          # Room DAO
    └── build.gradle.kts
```

## 1. DAO Layer (Database Access)

### Room DAO Implementation
```kotlin
// datasource/daos/ItemDao.kt
package com.project.feature.datasource.daos

import androidx.room.*
import kotlinx.coroutines.flow.Flow

@Dao
abstract class ItemDao {

    // Basic CRUD operations
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    abstract suspend fun createOrUpdate(vararg items: ItemDbDto)

    @Query("SELECT * FROM items WHERE id = :itemId")
    abstract suspend fun getItemById(itemId: String): ItemDbDto?

    @Query("SELECT * FROM items")
    abstract fun observeAll(): Flow<List<ItemDbDto>>

    @Query("DELETE FROM items WHERE id = :itemId")
    abstract suspend fun deleteById(itemId: String)

    @Query("DELETE FROM items")
    abstract suspend fun deleteAll()

    // Complex queries with joins
    @Transaction
    @Query("""
        SELECT i.*, u.name as userName 
        FROM items i 
        JOIN users u ON i.userId = u.id 
        WHERE i.category = :category
        ORDER BY i.createdAt DESC
    """)
    abstract fun observeItemsWithUserByCategory(
        category: String
    ): Flow<List<ItemWithUserDbDto>>

    // Transaction-based operations
    @Transaction
    open suspend fun replaceAll(vararg items: ItemDbDto) {
        deleteAll()
        createOrUpdate(*items)
    }

    // Batch operations
    @Transaction
    open suspend fun createOrUpdateWithRelations(
        item: ItemDbDto,
        tags: List<TagDbDto>,
        metadata: List<MetadataDbDto>
    ) {
        createOrUpdate(item)
        createOrUpdateTags(*tags.toTypedArray())
        createOrUpdateMetadata(*metadata.toTypedArray())
        
        // Clear existing relations
        deleteItemTagsByItemId(item.id)
        
        // Create new relations
        val itemTags = tags.map { 
            ItemXTagDbDto(itemId = item.id, tagId = it.id) 
        }
        createOrUpdateItemTags(*itemTags.toTypedArray())
    }

    // Supporting methods for relations
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    abstract suspend fun createOrUpdateTags(vararg tags: TagDbDto)

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    abstract suspend fun createOrUpdateMetadata(vararg metadata: MetadataDbDto)

    @Insert(onConflict = OnConflictStrategy.IGNORE)
    abstract suspend fun createOrUpdateItemTags(vararg itemTags: ItemXTagDbDto)

    @Query("DELETE FROM item_x_tags WHERE itemId = :itemId")
    abstract suspend fun deleteItemTagsByItemId(itemId: String)

    // Reactive queries with parameters
    fun observeItemById(itemId: String): Flow<ItemWithRelationsDbDto?> {
        return combine(
            observeItemByIdInternal(itemId),
            observeTagsByItemId(itemId),
            observeMetadataByItemId(itemId)
        ) { item, tags, metadata ->
            item?.copy(
                tags = tags,
                metadata = metadata
            )
        }
    }

    @Query("SELECT * FROM items WHERE id = :itemId")
    abstract fun observeItemByIdInternal(itemId: String): Flow<ItemDbDto?>

    @Query("""
        SELECT t.* FROM tags t 
        JOIN item_x_tags ixt ON t.id = ixt.tagId 
        WHERE ixt.itemId = :itemId
    """)
    abstract fun observeTagsByItemId(itemId: String): Flow<List<TagDbDto>>

    @Query("SELECT * FROM metadata WHERE itemId = :itemId")
    abstract fun observeMetadataByItemId(itemId: String): Flow<List<MetadataDbDto>>

    // Performance optimized queries
    @Query("SELECT id FROM items") 
    abstract suspend fun getAllItemIds(): List<String>

    @Query("""
        SELECT * FROM items 
        WHERE lastModified > :timestamp 
        ORDER BY lastModified DESC 
        LIMIT :limit
    """)
    abstract suspend fun getRecentlyModified(
        timestamp: Long, 
        limit: Int = 50
    ): List<ItemDbDto>
}
```

### Database Entity DTOs
```kotlin
// infrastructure/models/ItemDbDto.kt
package com.project.feature.infrastructure.models

import androidx.room.*
import com.project.models.enums.ItemType

@Entity(
    tableName = "items",
    indices = [
        Index(value = ["userId"]),
        Index(value = ["category"]),
        Index(value = ["createdAt"])
    ]
)
data class ItemDbDto(
    @PrimaryKey val id: String,
    val title: String,
    val description: String?,
    val userId: String,
    val category: String,
    val type: ItemType,
    val createdAt: Long,
    val lastModified: Long,
    val isActive: Boolean = true,
    
    // JSON column for complex data
    @ColumnInfo(name = "metadata_json")
    val metadataJson: String? = null,
    
    // Embedded value objects
    @Embedded(prefix = "location_")
    val location: LocationDbDto? = null
)

@Entity(
    tableName = "tags",
    indices = [Index(value = ["name"], unique = true)]
)
data class TagDbDto(
    @PrimaryKey val id: String,
    val name: String,
    val color: String? = null
)

@Entity(
    tableName = "item_x_tags",
    primaryKeys = ["itemId", "tagId"],
    foreignKeys = [
        ForeignKey(
            entity = ItemDbDto::class,
            parentColumns = ["id"],
            childColumns = ["itemId"],
            onDelete = ForeignKey.CASCADE
        ),
        ForeignKey(
            entity = TagDbDto::class,
            parentColumns = ["id"],
            childColumns = ["tagId"],
            onDelete = ForeignKey.CASCADE
        )
    ]
)
data class ItemXTagDbDto(
    val itemId: String,
    val tagId: String,
    val createdAt: Long = System.currentTimeMillis()
)

// Relation DTOs for complex queries
data class ItemWithUserDbDto(
    @Embedded val item: ItemDbDto,
    val userName: String
)

data class ItemWithRelationsDbDto(
    @Embedded val item: ItemDbDto,
    @Relation(
        parentColumn = "id",
        entityColumn = "itemId"
    )
    val tags: List<TagDbDto> = emptyList(),
    
    @Relation(
        parentColumn = "id",
        entityColumn = "itemId"
    )
    val metadata: List<MetadataDbDto> = emptyList()
)
```

## 2. DataSource Layer (Abstraction Interface)

### DataSource Interface
```kotlin
// infrastructure/FeatureDbDataSource.kt
package com.project.feature.infrastructure

import com.project.feature.infrastructure.models.*
import kotlinx.coroutines.flow.Flow

interface FeatureDbDataSource {
    
    // Basic operations
    suspend fun saveItems(vararg items: ItemDbDto)
    suspend fun getItemById(itemId: String): ItemDbDto?
    suspend fun deleteItem(itemId: String)
    suspend fun deleteItems(itemIds: List<String>)
    
    // Reactive queries
    fun observeAllItems(): Flow<List<ItemWithRelationsDbDto>>
    fun observeItemById(itemId: String): Flow<ItemWithRelationsDbDto?>
    fun observeItemsByCategory(category: String): Flow<List<ItemDbDto>>
    fun observeItemsByUser(userId: String): Flow<List<ItemDbDto>>
    
    // Complex operations
    suspend fun replaceAllItems(vararg items: ItemDbDto)
    suspend fun createItemWithRelations(
        item: ItemDbDto,
        tags: List<TagDbDto>,
        metadata: List<MetadataDbDto>
    )
    
    // Batch operations
    suspend fun getItemsByIds(itemIds: Set<String>): List<ItemDbDto>
    suspend fun getRecentlyModified(timestamp: Long, limit: Int): List<ItemDbDto>
    
    // Relations management
    suspend fun addTagsToItem(itemId: String, tagIds: List<String>)
    suspend fun removeTagsFromItem(itemId: String, tagIds: List<String>)
    
    // Utility operations
    suspend fun getAllItemIds(): List<String>
    suspend fun getItemsCount(): Int
    suspend fun getItemsCountByCategory(): Map<String, Int>
}
```

### DataSource Implementation
```kotlin
// datasource/FeatureDbDataSourceImpl.kt
package com.project.feature.datasource

import com.project.feature.datasource.daos.ItemDao
import com.project.feature.infrastructure.FeatureDbDataSource
import com.project.feature.infrastructure.models.*
import kotlinx.coroutines.flow.Flow
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class FeatureDbDataSourceImpl @Inject constructor(
    private val itemDao: ItemDao
) : FeatureDbDataSource {

    override suspend fun saveItems(vararg items: ItemDbDto) {
        itemDao.createOrUpdate(*items)
    }

    override suspend fun getItemById(itemId: String): ItemDbDto? {
        return itemDao.getItemById(itemId)
    }

    override suspend fun deleteItem(itemId: String) {
        itemDao.deleteById(itemId)
    }

    override suspend fun deleteItems(itemIds: List<String>) {
        itemIds.forEach { itemDao.deleteById(it) }
    }

    override fun observeAllItems(): Flow<List<ItemWithRelationsDbDto>> {
        return itemDao.observeAllItemsWithRelations()
    }

    override fun observeItemById(itemId: String): Flow<ItemWithRelationsDbDto?> {
        return itemDao.observeItemById(itemId)
    }

    override fun observeItemsByCategory(category: String): Flow<List<ItemDbDto>> {
        return itemDao.observeItemsByCategory(category)
    }

    override fun observeItemsByUser(userId: String): Flow<List<ItemDbDto>> {
        return itemDao.observeItemsByUser(userId)
    }

    override suspend fun replaceAllItems(vararg items: ItemDbDto) {
        itemDao.replaceAll(*items)
    }

    override suspend fun createItemWithRelations(
        item: ItemDbDto,
        tags: List<TagDbDto>,
        metadata: List<MetadataDbDto>
    ) {
        itemDao.createOrUpdateWithRelations(item, tags, metadata)
    }

    override suspend fun getItemsByIds(itemIds: Set<String>): List<ItemDbDto> {
        return itemDao.getItemsByIds(itemIds)
    }

    override suspend fun getRecentlyModified(timestamp: Long, limit: Int): List<ItemDbDto> {
        return itemDao.getRecentlyModified(timestamp, limit)
    }

    override suspend fun addTagsToItem(itemId: String, tagIds: List<String>) {
        val itemTags = tagIds.map { ItemXTagDbDto(itemId = itemId, tagId = it) }
        itemDao.createOrUpdateItemTags(*itemTags.toTypedArray())
    }

    override suspend fun removeTagsFromItem(itemId: String, tagIds: List<String>) {
        itemDao.deleteItemTagsByItemIdAndTagIds(itemId, tagIds)
    }

    override suspend fun getAllItemIds(): List<String> {
        return itemDao.getAllItemIds()
    }

    override suspend fun getItemsCount(): Int {
        return itemDao.getItemsCount()
    }

    override suspend fun getItemsCountByCategory(): Map<String, Int> {
        return itemDao.getItemsCountByCategory()
    }
}
```

## 3. Repository Layer (Business Logic Coordination)

### Repository Interface
```kotlin
// domain/FeatureRepository.kt
package com.project.feature.domain

import com.project.models.valueobjects.AuthHeader
import com.project.feature.domain.models.*
import kotlinx.coroutines.flow.Flow

interface FeatureRepository {
    
    // Remote operations
    suspend fun fetchItems(
        header: AuthHeader,
        userId: String,
        category: String? = null
    ): List<Item>
    
    suspend fun createItem(
        header: AuthHeader, 
        item: Item
    ): Item
    
    suspend fun updateItem(
        header: AuthHeader,
        item: Item
    ): Item
    
    suspend fun deleteItem(
        header: AuthHeader,
        itemId: String
    )
    
    // Local operations
    suspend fun saveLocalItems(vararg items: Item)
    suspend fun deleteLocalItem(itemId: String)
    suspend fun replaceLocalItems(vararg items: Item)
    
    // Reactive local queries
    fun observeLocalItems(): Flow<List<Item>>
    fun observeLocalItemById(itemId: String): Flow<Item?>
    fun observeLocalItemsByCategory(category: String): Flow<List<Item>>
    
    // Hybrid operations (remote + local)
    suspend fun refreshItems(
        header: AuthHeader,
        userId: String
    )
    
    suspend fun syncItem(
        header: AuthHeader,
        item: Item
    ): Item
    
    // Utility operations
    suspend fun getLocalItemIds(): List<String>
    suspend fun getItemsRequiringSync(): List<Item>
}
```

### Repository Implementation
```kotlin
// infrastructure/FeatureRepositoryImpl.kt
package com.project.feature.infrastructure

import com.project.basedomain.usecases.UseCaseResult
import com.project.basedomain.usecases.mapToUseCaseResult
import com.project.basedomain.usecases.whenSuccessSuspending
import com.project.feature.domain.FeatureRepository
import com.project.feature.domain.models.Item
import com.project.feature.infrastructure.converters.*
import com.project.safethriftclients.SafeTFeatureService
import com.project.models.valueobjects.AuthHeader
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class FeatureRepositoryImpl @Inject constructor(
    private val featureService: SafeTFeatureService,
    private val featureDbDataSource: FeatureDbDataSource,
) : FeatureRepository {

    override suspend fun fetchItems(
        header: AuthHeader,
        userId: String,
        category: String?
    ): List<Item> {
        val response = featureService.getItems(
            header = header.convertToTAuthHeader(),
            request = TGetItemsRequest(
                userId = userId,
                category = category
            )
        )
        return response.items.map { it.convertToItem() }
    }

    override suspend fun createItem(
        header: AuthHeader,
        item: Item
    ): Item {
        val response = featureService.createItem(
            header = header.convertToTAuthHeader(),
            request = TCreateItemRequest(
                item = item.convertToTItem()
            )
        )
        val createdItem = response.item.convertToItem()
        // Update local cache
        saveLocalItems(createdItem)
        return createdItem
    }

    override suspend fun updateItem(
        header: AuthHeader,
        item: Item
    ): Item {
        val response = featureService.updateItem(
            header = header.convertToTAuthHeader(),
            request = TUpdateItemRequest(
                item = item.convertToTItem()
            )
        )
        val updatedItem = response.item.convertToItem()
        // Update local cache
        saveLocalItems(updatedItem)
        return updatedItem
    }

    override suspend fun deleteItem(
        header: AuthHeader,
        itemId: String
    ) {
        featureService.deleteItem(
            header = header.convertToTAuthHeader(),
            request = TDeleteItemRequest(itemId = itemId)
        )
        // Remove from local cache
        deleteLocalItem(itemId)
    }

    override suspend fun saveLocalItems(vararg items: Item) {
        featureDbDataSource.saveItems(
            *items.map { it.convertToDbDto() }.toTypedArray()
        )
    }

    override suspend fun deleteLocalItem(itemId: String) {
        featureDbDataSource.deleteItem(itemId)
    }

    override suspend fun replaceLocalItems(vararg items: Item) {
        featureDbDataSource.replaceAllItems(
            *items.map { it.convertToDbDto() }.toTypedArray()
        )
    }

    override fun observeLocalItems(): Flow<List<Item>> {
        return featureDbDataSource.observeAllItems().map { itemsWithRelations ->
            itemsWithRelations.map { it.convertToItem() }
        }
    }

    override fun observeLocalItemById(itemId: String): Flow<Item?> {
        return featureDbDataSource.observeItemById(itemId).map { itemWithRelations ->
            itemWithRelations?.convertToItem()
        }
    }

    override fun observeLocalItemsByCategory(category: String): Flow<List<Item>> {
        return featureDbDataSource.observeItemsByCategory(category).map { items ->
            items.map { it.convertToItem() }
        }
    }

    override suspend fun refreshItems(
        header: AuthHeader,
        userId: String
    ) {
        val items = fetchItems(header, userId)
        replaceLocalItems(*items.toTypedArray())
    }

    override suspend fun syncItem(
        header: AuthHeader,
        item: Item
    ): Item {
        return if (item.isLocalOnly) {
            createItem(header, item)
        } else {
            updateItem(header, item)
        }
    }

    override suspend fun getLocalItemIds(): List<String> {
        return featureDbDataSource.getAllItemIds()
    }

    override suspend fun getItemsRequiringSync(): List<Item> {
        val timestamp = System.currentTimeMillis() - (24 * 60 * 60 * 1000) // 24 hours ago
        return featureDbDataSource.getRecentlyModified(timestamp, 100)
            .map { it.convertToItem() }
            .filter { it.needsSync }
    }
}
```

## 4. Type Converters and Data Transformation

### Domain to Database Converters
```kotlin
// infrastructure/converters/DomainToDbConverters.kt
package com.project.feature.infrastructure.converters

import com.project.feature.domain.models.Item
import com.project.feature.infrastructure.models.ItemDbDto
import com.google.gson.Gson

private val gson = Gson()

fun Item.convertToDbDto(): ItemDbDto {
    return ItemDbDto(
        id = this.id,
        title = this.title,
        description = this.description,
        userId = this.userId,
        category = this.category,
        type = this.type,
        createdAt = this.createdAt,
        lastModified = this.lastModified,
        isActive = this.isActive,
        metadataJson = this.metadata?.let { gson.toJson(it) },
        location = this.location?.convertToDbDto()
    )
}

fun ItemDbDto.convertToItem(): Item {
    return Item(
        id = this.id,
        title = this.title,
        description = this.description,
        userId = this.userId,
        category = this.category,
        type = this.type,
        createdAt = this.createdAt,
        lastModified = this.lastModified,
        isActive = this.isActive,
        metadata = this.metadataJson?.let { 
            gson.fromJson(it, Map::class.java) as Map<String, Any>
        },
        location = this.location?.convertToLocation()
    )
}

fun ItemWithRelationsDbDto.convertToItem(): Item {
    return this.item.convertToItem().copy(
        tags = this.tags.map { it.convertToTag() },
        metadata = this.metadata.map { it.convertToMetadata() }
    )
}
```

### Room Type Converters
```kotlin
// database/converters/DatabaseConverters.kt
package com.project.database.converters

import androidx.room.TypeConverter
import com.project.models.enums.ItemType
import com.google.gson.Gson
import com.google.gson.reflect.TypeToken

class DatabaseConverters {
    
    private val gson = Gson()
    
    @TypeConverter
    fun fromItemType(value: ItemType): String {
        return value.name
    }
    
    @TypeConverter
    fun toItemType(value: String): ItemType {
        return ItemType.valueOf(value)
    }
    
    @TypeConverter
    fun fromStringList(value: List<String>?): String? {
        return value?.let { gson.toJson(it) }
    }
    
    @TypeConverter
    fun toStringList(value: String?): List<String>? {
        return value?.let {
            val listType = object : TypeToken<List<String>>() {}.type
            gson.fromJson(it, listType)
        }
    }
    
    @TypeConverter
    fun fromStringMap(value: Map<String, String>?): String? {
        return value?.let { gson.toJson(it) }
    }
    
    @TypeConverter
    fun toStringMap(value: String?): Map<String, String>? {
        return value?.let {
            val mapType = object : TypeToken<Map<String, String>>() {}.type
            gson.fromJson(it, mapType)
        }
    }
}
```

## 5. Database Module Configuration

### Room Database Setup
```kotlin
// database/ProjectDatabase.kt
package com.project.database

import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.room.TypeConverters
import androidx.room.migration.Migration
import androidx.sqlite.db.SupportSQLiteDatabase
import android.content.Context
import com.project.database.converters.DatabaseConverters
import com.project.feature.datasource.daos.*
import com.project.feature.infrastructure.models.*

@Database(
    entities = [
        ItemDbDto::class,
        TagDbDto::class,
        ItemXTagDbDto::class,
        MetadataDbDto::class,
        UserDbDto::class
    ],
    version = 5,
    exportSchema = true
)
@TypeConverters(DatabaseConverters::class)
abstract class ProjectDatabase : RoomDatabase() {
    
    // DAOs
    abstract fun itemDao(): ItemDao
    abstract fun tagDao(): TagDao
    abstract fun userDao(): UserDao
    
    companion object {
        const val DATABASE_NAME = "project_database.db"
        
        fun create(context: Context): ProjectDatabase {
            return Room.databaseBuilder(
                context,
                ProjectDatabase::class.java,
                DATABASE_NAME
            )
            .addMigrations(MIGRATION_1_2, MIGRATION_2_3, MIGRATION_3_4, MIGRATION_4_5)
            .build()
        }
        
        private val MIGRATION_1_2 = object : Migration(1, 2) {
            override fun migrate(database: SupportSQLiteDatabase) {
                database.execSQL("""
                    ALTER TABLE items 
                    ADD COLUMN metadata_json TEXT
                """)
            }
        }
        
        private val MIGRATION_2_3 = object : Migration(2, 3) {
            override fun migrate(database: SupportSQLiteDatabase) {
                database.execSQL("""
                    CREATE TABLE IF NOT EXISTS tags (
                        id TEXT PRIMARY KEY NOT NULL,
                        name TEXT NOT NULL,
                        color TEXT
                    )
                """)
                
                database.execSQL("""
                    CREATE UNIQUE INDEX IF NOT EXISTS index_tags_name 
                    ON tags (name)
                """)
            }
        }
    }
}
```

## 6. Dependency Injection Setup

### DI Module for Data Layer
```kotlin
// di/DataSourceModule.kt
package com.project.di

import android.content.Context
import androidx.room.Room
import com.project.database.ProjectDatabase
import com.project.feature.datasource.*
import com.project.feature.infrastructure.*
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DataSourceModule {
    
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): ProjectDatabase {
        return ProjectDatabase.create(context)
    }
    
    @Provides
    fun provideItemDao(database: ProjectDatabase): ItemDao {
        return database.itemDao()
    }
    
    @Provides
    @Singleton
    fun provideFeatureDbDataSource(
        itemDao: ItemDao
    ): FeatureDbDataSource {
        return FeatureDbDataSourceImpl(itemDao)
    }
}

// di/RepositoryModule.kt
@Module
@InstallIn(SingletonComponent::class)
object RepositoryModule {
    
    @Provides
    @Singleton
    fun provideFeatureRepository(
        featureService: SafeTFeatureService,
        featureDbDataSource: FeatureDbDataSource
    ): FeatureRepository {
        return FeatureRepositoryImpl(featureService, featureDbDataSource)
    }
}
```

## 7. Testing Strategy

### DAO Testing
```kotlin
class ItemDaoTest {
    
    @get:Rule
    val instantTaskExecutorRule = InstantTaskExecutorRule()
    
    private lateinit var database: ProjectDatabase
    private lateinit var itemDao: ItemDao
    
    @Before
    fun setup() {
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            ProjectDatabase::class.java
        ).allowMainThreadQueries().build()
        
        itemDao = database.itemDao()
    }
    
    @After
    fun teardown() {
        database.close()
    }
    
    @Test
    fun `createOrUpdate should insert new item`() = runTest {
        // Given
        val item = createTestItem()
        
        // When
        itemDao.createOrUpdate(item)
        
        // Then
        val result = itemDao.getItemById(item.id)
        assertThat(result).isEqualTo(item)
    }
    
    @Test
    fun `observeAll should emit updates`() = runTest {
        // Given
        val items = listOf(createTestItem("1"), createTestItem("2"))
        
        // When
        val flow = itemDao.observeAll()
        val testObserver = flow.test()
        
        itemDao.createOrUpdate(*items.toTypedArray())
        
        // Then
        testObserver.assertValues(
            emptyList(),
            items
        )
    }
}
```

### Repository Testing
```kotlin
class FeatureRepositoryImplTest {
    
    @MockK lateinit var featureService: SafeTFeatureService
    @MockK lateinit var featureDbDataSource: FeatureDbDataSource
    
    private lateinit var repository: FeatureRepositoryImpl
    
    @Before
    fun setup() {
        repository = FeatureRepositoryImpl(featureService, featureDbDataSource)
    }
    
    @Test
    fun `fetchItems should return items list`() = runTest {
        // Given
        val header = createTestAuthHeader()
        val userId = "user123"
        val expectedItems = listOf(createTestItem())
        
        coEvery { 
            featureService.getItems(any(), any()) 
        } returns TGetItemsResponse(items = expectedItems.map { it.convertToTItem() })
        
        // When
        val result = repository.fetchItems(header, userId)
        
        // Then
        assertThat(result).isEqualTo(expectedItems)
        coVerify { featureService.getItems(header.convertToTAuthHeader(), any()) }
    }
}
```

## 8. Best Practices

### Do's ✅
- Use `@Transaction` for operations that modify multiple tables
- Implement proper foreign key relationships with cascade rules
- Use Flow for reactive queries that need real-time updates
- Separate domain models from database DTOs
- Use type converters for complex data types
- Implement proper database migrations
- Use indices for frequently queried columns
- Handle conflict resolution strategies appropriately
- Test DAOs with in-memory databases
- Use suspend functions for database operations

### Don'ts ❌
- Don't expose database DTOs to the domain layer
- Don't perform database operations on the main thread
- Don't ignore database migration strategies
- Don't forget to handle foreign key constraints
- Don't use raw queries when Room annotations suffice
- Don't skip testing database operations
- Don't ignore transaction boundaries for related operations
- Don't forget to close database connections in tests

This architecture ensures a clean, testable, and maintainable data layer that properly separates concerns while providing excellent performance and type safety.