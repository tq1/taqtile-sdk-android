  1. Download the Taqtile Android SDK jar file from our GitHub repository [here](https://github.com/shingle/taqtile-sdk-android/releases/latest);
  2. Add this file as a gradle dependency, replacing <version> with the version number downloaded on step 1 (e.g, *1.3.4*).

```java
compile "com.taqtile:taqtile-sdk:<Version Here>"
```
If your application targets Android versions 14+, you can also add the SDK as a gradle dependency directly:

```java
compile 'com.taqtile:taqtile-sdk-android:<Version Here>'
```

If you are using an SDK version superior than 2.8.0, you should also add the location of Esri's maven repository to the main application *build.gradle* file.

```java
allprojects {
  repositories {
    jcenter()
    maven {
      url 'http://dl.bintray.com/esri/android'
    }
  }
}
```
