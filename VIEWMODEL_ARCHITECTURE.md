# ViewModel Architecture Guide

This document defines the standardized approach for creating ViewModels in our Android project, based on analysis of the Sunshine-Photos architecture.

## Core Principles

1. **Single Responsibility**: Each ViewModel manages one specific UI concern
2. **Reactive State Management**: Use StateFlow for reactive UI updates
3. **Event-Driven Navigation**: Handle navigation through sealed class events
4. **Clean Architecture**: Interact with domain layer through use cases
5. **Proper Threading**: Use coroutines with appropriate dispatchers

## ViewModel Template Structure

```kotlin
@HiltViewModel
class [Feature]ViewModel @Inject constructor(
    // Dependencies injected here
    private val performSomeActionUseCase: PerformSomeActionUseCase,
    private val observeDataUseCase: ObserveDataUseCase,
) : ViewModel() {

    // Internal state management
    private val _state = MutableStateFlow([Feature]State())
    val state = _state.asStateFlow()

    // Navigation events
    private val _events = Channel<Event>()
    val events = _events.receiveAsFlow()

    // Exposed reactive data
    val someData = observeDataUseCase()
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), emptyList())

    init {
        // Initialize data collection and event listeners
        initializeData()
        setupEventListeners()
    }

    // Public action methods
    fun performAction() {
        viewModelScope.launch(Dispatchers.IO) {
            _state.update { it.copy(isLoading = true) }
            
            try {
                val result = performSomeActionUseCase()
                handleSuccess(result)
            } catch (error: Exception) {
                handleError(error)
            }
                
            _state.update { it.copy(isLoading = false) }
        }
    }

    // Private helper methods
    private fun handleSuccess(data: SomeResult) {
        viewModelScope.launch {
            _events.send(Event.NavigateToNext(data.id))
        }
    }

    private fun handleError(error: Throwable) {
        _state.update { 
            it.copy(errorMessage = error.reasonOrMessage()) 
        }
    }

    // Sealed class for navigation events
    sealed class Event {
        data class NavigateToNext(val id: String) : Event()
        data object NavigateBack : Event()
    }
}
```

## State Data Class Pattern

```kotlin
data class [Feature]State(
    val isLoading: Boolean = false,
    val isRefreshing: Boolean = false,
    val errorMessage: String? = null,
    val bottomMessage: Message = Message(),
    // Feature-specific state properties
    val searchText: String = "",
    val selectedItems: List<Item> = emptyList(),
    val showDialog: Boolean = false,
)
```

## Required Dependencies

### Essential Use Cases
```kotlin
// For data observation
private val observeDataUseCase: ObserveDataUseCase

// For actions
private val performActionUseCase: PerformActionUseCase

// For refresh operations
private val refreshDataUseCase: RefreshDataUseCase
```

### Optional Dependencies
```kotlin
// For logging
private val logUseCase: LogUseCase

// For analytics
private val logAnalyticsEventUseCase: LogAnalyticsEventUseCase

// For background operations
private val globalIOCoroutineScope: CoroutineScope
```

## ViewModel Types and Patterns

### 1. List/Collection ViewModels
**Use for**: Home screens, galleries, contact lists

```kotlin
@HiltViewModel
class ItemsListViewModel @Inject constructor(
    private val observeItemsUseCase: ObserveItemsUseCase,
    private val refreshItemsUseCase: RefreshItemsUseCase,
) : ViewModel() {

    private val _state = MutableStateFlow(ItemsListState())
    val state = _state.asStateFlow()

    private val _searchText = MutableStateFlow("")
    val searchText = _searchText.asStateFlow()

    // Reactive filtered data
    val itemsFiltered = combine(observeItemsUseCase(), _searchText) { items, search ->
        if (search.isEmpty()) items
        else items.filter { it.title.contains(search, ignoreCase = true) }
    }.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), emptyList())

    fun swipeToRefresh() {
        viewModelScope.launch(Dispatchers.IO) {
            _state.update { it.copy(isRefreshing = true) }
            try {
                refreshItemsUseCase()
            } catch (error: Exception) {
                handleError(error)
            }
            _state.update { it.copy(isRefreshing = false) }
        }
    }

    fun setSearchText(text: String) {
        _searchText.update { text }
    }
}
```

### 2. Authentication ViewModels
**Use for**: Login, signup, verification flows

```kotlin
@HiltViewModel
class LoginViewModel @Inject constructor(
    private val authenticateUserUseCase: AuthenticateUserUseCase,
    private val sendVerificationCodeUseCase: SendVerificationCodeUseCase,
) : ViewModel() {

    private val _state = MutableStateFlow(LoginState())
    val state = _state.asStateFlow()

    private var countdownJob: Job? = null

    fun signInWithCredentials(email: String, password: String) {
        viewModelScope.launch(Dispatchers.IO) {
            _state.update { it.copy(isLoading = true, errorMessage = null) }
            
            try {
                val result = authenticateUserUseCase(email, password)
                navigateToMain()
            } catch (error: Exception) {
                processAuthError(error)
            }
        }
    }

    private fun startCountdown(seconds: Int) {
        countdownJob?.cancel()
        countdownJob = viewModelScope.launch {
            for (i in seconds downTo 0) {
                _state.update { it.copy(countdownSeconds = i) }
                delay(1000L)
            }
        }
    }
}
```

### 3. Detail ViewModels
**Use for**: Item details, profile screens, settings

```kotlin
@HiltViewModel
class ItemDetailViewModel @Inject constructor(
    private val observeItemUseCase: ObserveItemUseCase,
    private val updateItemUseCase: UpdateItemUseCase,
    private val deleteItemUseCase: DeleteItemUseCase,
) : ViewModel() {

    private val _state = MutableStateFlow(ItemDetailState())
    val state = _state.asStateFlow()

    fun loadItem(itemId: String) {
        viewModelScope.launch {
            observeItemUseCase(itemId)
                .collect { item ->
                    _state.update { it.copy(item = item) }
                }
        }
    }

    fun deleteItem() {
        viewModelScope.launch(Dispatchers.IO) {
            _state.update { it.copy(isLoading = true) }
            
            try {
                deleteItemUseCase(state.value.item?.id.orEmpty())
                _events.send(Event.NavigateBack)
            } catch (error: Exception) {
                handleError(error)
            }
        }
    }
}
```

### 4. Form/Creation ViewModels
**Use for**: Create item, edit forms, multi-step wizards

```kotlin
@HiltViewModel
class CreateItemViewModel @Inject constructor(
    private val createItemUseCase: CreateItemUseCase,
    private val validateInputUseCase: ValidateInputUseCase,
) : ViewModel() {

    private val _state = MutableStateFlow(CreateItemState())
    val state = _state.asStateFlow()

    fun setTitle(title: String) {
        _state.update { it.copy(title = title) }
        validateForm()
    }

    fun addSelectedItem(item: SelectableItem) {
        val updated = _state.value.selectedItems + item
        _state.update { it.copy(selectedItems = updated) }
    }

    fun removeSelectedItem(item: SelectableItem) {
        val updated = _state.value.selectedItems.filterNot { it.id == item.id }
        _state.update { it.copy(selectedItems = updated) }
    }

    fun createItem() {
        if (!isFormValid()) return
        
        viewModelScope.launch(Dispatchers.IO) {
            _state.update { it.copy(isLoading = true) }
            
            try {
                val item = createItemUseCase(
                    title = _state.value.title,
                    selectedItems = _state.value.selectedItems
                )
                _events.send(Event.NavigateToItem(item.id))
            } catch (error: Exception) {
                handleError(error)
            }
        }
    }

    private fun isFormValid(): Boolean {
        return _state.value.title.isNotBlank() && 
               _state.value.selectedItems.isNotEmpty()
    }
}
```

## Common Patterns

### 1. Error Handling
```kotlin
private fun handleError(error: Throwable) {
    viewModelScope.launch {
        logUseCase(LogLevel.ERROR, TAG, "operation_failed", error)
    }
    
    _state.update { 
        it.copy(
            isLoading = false,
            bottomMessage = ErrorMessage(
                error.reasonOrMessage()?.let { UiText.DynamicString(it) }
                    ?: UiText.StringResource(R.string.generic_error)
            )
        )
    }
}
```

### 2. External Event Listening
```kotlin
private fun listenToExternalEvents() {
    viewModelScope.launch(Dispatchers.IO) {
        eventFlow.javaCompatibilityListen(SomeEvent::class.java)
            .collect { event ->
                handleExternalEvent(event.data)
            }
    }
}
```

### 3. Background Operations
```kotlin
private fun performBackgroundOperation() {
    globalIOCoroutineScope.launch {
        // Long-running operation that shouldn't be cancelled with ViewModel
        backgroundUseCase()
    }
}
```

### 4. Flow Combination
```kotlin
val combinedData = combine(
    observeDataAUseCase(),
    observeDataBUseCase(),
    _state
) { dataA, dataB, state ->
    CombinedData(dataA, dataB, state.filter)
}.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), CombinedData())
```

## Best Practices

### Do's ✅
- Always use `@HiltViewModel` annotation
- Use `MutableStateFlow` for mutable state
- Expose immutable flows with `asStateFlow()`
- Handle loading states consistently
- Use sealed classes for navigation events
- Validate inputs before processing
- Log errors with context
- Use appropriate coroutine dispatchers
- Cancel long-running operations in `onCleared()`

### Don'ts ❌
- Don't access UI components directly
- Don't perform blocking operations on Main dispatcher
- Don't forget to handle error states
- Don't expose mutable state directly
- Don't use global state when local state suffices
- Don't forget to cancel countdowns/timers
- Don't ignore external event listeners cleanup

## Testing Guidelines

```kotlin
class ItemViewModelTest {
    
    @MockK lateinit var observeItemsUseCase: ObserveItemsUseCase
    @MockK lateinit var refreshItemsUseCase: RefreshItemsUseCase
    
    private lateinit var viewModel: ItemViewModel
    
    @Test
    fun `when refresh is called, loading state is updated correctly`() = runTest {
        // Given
        coEvery { refreshItemsUseCase() } returns Unit
        
        // When
        viewModel.refresh()
        
        // Then
        assertThat(viewModel.state.value.isLoading).isFalse()
        coVerify { refreshItemsUseCase() }
    }
}
```

## File Structure

```
feature/
├── viewmodel/
│   ├── build.gradle.kts
│   └── src/main/java/
│       └── com/project/feature/viewmodel/
│           ├── FeatureViewModel.kt
│           ├── FeatureState.kt
│           └── util/
│               └── FeatureUtils.kt
```

This architecture ensures consistent, testable, and maintainable ViewModels across the entire project.