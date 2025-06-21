# Architecture Updates Summary

This document summarizes the changes made to simplify the use case pattern and remove the Command pattern dependency.

## Changes Made

### 1. VIEWMODEL_ARCHITECTURE.md Updates

**Before:**
```kotlin
private val someUseCase: SuspendUseCase<SomeCommand, SomeResult>
private val observeUseCase: UseCase<ObserveCommand, Flow<SomeData>>

// Usage
someUseCase(SomeCommand()).whenSuccess { ... }.whenError { ... }
```

**After:**
```kotlin
private val performSomeActionUseCase: PerformSomeActionUseCase
private val observeDataUseCase: ObserveDataUseCase

// Usage
try {
    val result = performSomeActionUseCase()
    handleSuccess(result)
} catch (error: Exception) {
    handleError(error)
}
```

### 2. DATA_LAYER_ARCHITECTURE.md Updates

**Before:**
```kotlin
suspend fun fetchItems(...): UseCaseResult<List<Item>>
suspend fun createItem(...): UseCaseResult<Item>
```

**After:**
```kotlin
suspend fun fetchItems(...): List<Item>
suspend fun createItem(...): Item
```

### 3. Key Simplifications

1. **Removed Command Pattern**: No more `SomeCommand` data classes
2. **Simplified Use Case Naming**: `[WhatItDoes]UseCase` instead of `SuspendUseCase<Command, Result>`
3. **Direct Exception Handling**: Use try/catch instead of `UseCaseResult.whenSuccess/whenError`
4. **Cleaner Method Signatures**: Return actual types instead of wrapped results
5. **Simpler Testing**: Mock use cases directly without command objects

## New Use Case Pattern

### Use Case Definition
```kotlin
class FetchItemsUseCase @Inject constructor(
    private val repository: ItemsRepository
) {
    suspend operator fun invoke(userId: String, category: String? = null): List<Item> {
        return repository.fetchItems(userId, category)
    }
}
```

### Use Case Usage in ViewModels
```kotlin
@HiltViewModel
class ItemsViewModel @Inject constructor(
    private val fetchItemsUseCase: FetchItemsUseCase,
    private val observeItemsUseCase: ObserveItemsUseCase,
) : ViewModel() {

    fun loadItems() {
        viewModelScope.launch(Dispatchers.IO) {
            _state.update { it.copy(isLoading = true) }
            
            try {
                val items = fetchItemsUseCase(userId = currentUserId)
                _state.update { it.copy(items = items, isLoading = false) }
            } catch (error: Exception) {
                handleError(error)
            }
        }
    }
}
```

### Testing Use Cases
```kotlin
class ItemsViewModelTest {
    @MockK lateinit var fetchItemsUseCase: FetchItemsUseCase
    
    @Test
    fun `loadItems should update state correctly`() = runTest {
        // Given
        val expectedItems = listOf(createTestItem())
        coEvery { fetchItemsUseCase(any()) } returns expectedItems
        
        // When
        viewModel.loadItems()
        
        // Then
        assertThat(viewModel.state.value.items).isEqualTo(expectedItems)
        coVerify { fetchItemsUseCase(currentUserId) }
    }
}
```

## Benefits of the New Approach

1. **Simpler**: No need for command classes for simple parameters
2. **More Readable**: Clear method names and direct parameter passing
3. **Less Boilerplate**: Fewer classes and interfaces to maintain
4. **Standard Kotlin**: Uses standard exception handling instead of custom result wrappers
5. **Easier Testing**: Direct mocking without wrapper objects
6. **Company Agnostic**: No dependency on specific shared libraries

## Migration Guidelines

When creating new use cases:

1. **Name**: Use `[ActionDescription]UseCase` format
2. **Invoke**: Always implement `suspend operator fun invoke(...)`
3. **Parameters**: Pass parameters directly, not wrapped in command objects
4. **Return Types**: Return actual domain types, not wrapped results
5. **Error Handling**: Let exceptions bubble up, handle in ViewModels with try/catch

This simplified approach maintains clean architecture principles while reducing complexity and improving developer experience.