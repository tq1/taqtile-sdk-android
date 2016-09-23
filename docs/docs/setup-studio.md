There are two ways in order to get our SDK

 - If you will have compatibility with Android 13-, download the Taqtile Android SDK jar file from our GitHub repository [here](https://github.com/tq1/taqtile-sdk-android/releases/latest) or add jcenter as a repository on the project's 'build.gradle';

  OR

 -  If yoour target is 14+, you can also add jcenter as a repository on the project's top-level 'build.gradle' file as in the following example:

 ```
 allprojects {
    repositories {
        jcenter()
    }
}
 ```


If you followed the first option, you must also add the the gradle dependency to the project's `build.gradle` file, replacing <version> with the version number downloaded on step 1 (e.g, *1.3.4*).

```java
compile "com.taqtile:taqtile-sdk:<Version Here>"
```
If your application targets Android versions 14+, you can also add the SDK as a gradle dependency directly:

```java
compile 'com.taqtile:taqtile-sdk-android:<Version Here>'
```

If you are using an SDK version superior than 2.7.0, you should also add the location of the geotrigger SDK maven repository to the main application *build.gradle* file.

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
