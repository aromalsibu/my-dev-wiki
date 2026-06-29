# Riverpod

* **Getting Started**
    * What is Riverpod?
    * Installation & Code Generation (@riverpod)
    * Your First Provider
    * Consumer Widgets
    * ProviderScope
* **Core Concepts**
    * ProviderContainer
    * ProviderScope
    * Provider Lifecycle
    * Dependency Graph
    * State Invalidation & Refresh
    * Caching
    * Auto Dispose
* **Provider Types**
    * Modern Architecture (Recommended)
        * Provider
        * NotifierProvider
        * AsyncNotifierProvider
        * StreamNotifierProvider
    * Classic Declarative Types
        * StateProvider
        * FutureProvider
        * StreamProvider
    * Legacy Providers (Avoid in New Code)
        * ChangeNotifierProvider (Legacy)
        * StateNotifierProvider (Legacy)
* **Notifiers & Mutations**
    * Notifier
    * AsyncNotifier
    * StreamNotifier
    * build()
    * state
    * ref
    * Provider Families
    * Executing Mutations & Side Effects
* **Ref**
    * What is Ref?
    * ref.watch()
    * ref.read()
    * ref.listen()
    * ref.listenManual()
    * ref.invalidate()
    * ref.invalidateSelf()
    * ref.refresh()
    * ref.keepAlive()
    * ref.onDispose()
    * ref.onCancel()
    * ref.onResume()
    * ref.listenSelf()
    * ref.exists()
    * Ref Lifecycle
* **AsyncValue**
    * What is AsyncValue?
    * Loading
    * Data
    * Error
    * when()
    * maybeWhen()
    * map()
    * maybeMap()
    * guard()
    * Best Practices
* **Provider Modifiers**
    * .autoDispose
    * .family
    * .overrideWith
    * .overrideWithValue
    * .select
    * keepAlive
* **Consumers**
    * Consumer
    * ConsumerWidget
    * ConsumerStatefulWidget
    * WidgetRef
* **Provider Observers**
    * ProviderObserver
    * Logging
    * Debugging
    * Performance Monitoring
* **Testing**
    * ProviderContainer
    * Overriding Providers
    * Testing Providers
    * Testing Notifiers
    * Mocking Dependencies
    * Integration Testing
* **Performance**
    * Minimizing Rebuilds
    * select()
    * Splitting Providers
    * Auto Dispose Strategy
    * Caching
    * Common Performance Pitfalls
* **Best Practices**
    * Project Structure
    * Naming Conventions
    * Provider Organization
    * Error Handling
    * State Management Patterns
    * Dependency Injection
* **Migration**
    * Provider Package → Riverpod
    * StateNotifier → Notifier
    * FutureProvider → AsyncNotifier
    * StateProvider → Notifier
    * Riverpod 2 → Riverpod 3
* **Comparisons**
    * Provider vs StateProvider
    * FutureProvider vs AsyncNotifier
    * StreamProvider vs StreamNotifier
    * Notifier vs AsyncNotifier
    * Notifier vs StateNotifier
    * Provider vs Bloc
    * Provider vs GetX
    * Choosing the Right Riverpod Provider
* **API Cheat Sheet**
    * Common APIs
    * Lifecycle APIs
    * Ref APIs
    * AsyncValue APIs
    * Provider Types Overview