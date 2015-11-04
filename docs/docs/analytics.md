# Custom Data

The application can pass to the API any custom information needed about the user through the method *addCustomContent*. Below follows an example implementation:

```java
Map<String, Object> meta = new HashMap<>();
meta.put("key1", "value_1");
meta.put("key2", 123);

// Fields marked as `ignore` can not be used as push segmentation.
// This should be used with user specific fields
ArrayList<String> ignore = new ArrayList<>();
ignore.add("key1");
TQAnalytics.shared().addCustomData(meta, ignore);
```

# Events

You can easily send to our API information of what screens the user has been navigating in your application (or any other kind of event you want). Just add the code below on every point of interest, replacing "Welcome" with any other identifier:

```java
Map<String, String> segmentation = new HashMap<>();
segmentation.put("Page name", "Welcome");
TQAnalytics.shared().recordEvent("Page Views", segmentation, 1);
```