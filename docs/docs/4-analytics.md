# Custom Data

The application can pass to the API any custom information (in the form of a String) needed about the user through the method *addCustomContent*. Those custom data can be used on our admin portal to send segmented push notifications and even to send individual exclusive notifications through our public api. Below follows an example implementation:

```java
Map<String, String> meta = new HashMap<>();
meta.put("key1", "value_1");
meta.put("key2", "value_2");

// Fields marked as `ignore` can not be used as push segmentation.
// This should be used with user specific fields
ArrayList<String> ignore = new ArrayList<>();
ignore.add("key1");
TQAnalytics.shared().addCustomData(meta, ignore);
```

# Events

You can easily send to our API any event you want based on app usage. Examples include screen views, payments concluded, accounts created via mobile and so on. Also you can associate this event with custom fields (segmentations). Later those events thrown are available in our admin portal under the `Analytics` menu. In order to implement events just follow the structure below on every point of interest:

```java
Map<String, String> segmentation = new HashMap<>();
segmentation.put("Page Name", "Welcome");
TQAnalytics.shared().recordEvent("Page Views", segmentation, 1);
```