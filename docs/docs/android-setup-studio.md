1. Download the Taqtile Android SDK jar file from our GitHub repository [here](https://github.com/shingle/taqtile-sdk-android/releases/latest);
2. Add this file as a gradle dependency, replacing <version> with the version number downloaded on step 1 (e.g, *1.3.4*).
[block:code]
{
  "codes": [
    {
      "code": "compile \"com.taqtile:taqtile-sdk:<Version Here>\"",
      "language": "java",
      "name": "Project/app/build.gradle"
    }
  ]
}
[/block]
If your application targets Android versions 14+, you can also add the SDK as a gradle dependency directly:
[block:code]
{
  "codes": [
    {
      "code": "compile 'com.taqtile:taqtile-sdk-android:<Version Here>'",
      "language": "java"
    }
  ]
}
[/block]
If you are using an SDK version superior than 2.8.0, you should also add the location of Esri's maven repository to the main application *build.gradle* file.
[block:code]
{
  "codes": [
    {
      "code": "allprojects {\n    repositories {\n        jcenter()\n        maven {\n            url 'http://dl.bintray.com/esri/android'\n        }\n    }\n}",
      "language": "java",
      "name": null
    }
  ]
}
[/block]