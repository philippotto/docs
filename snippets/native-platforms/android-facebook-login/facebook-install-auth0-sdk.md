In your Android application, add this line to the `app/build.gradle` file

```gradle
dependencies {
    implementation 'com.auth0.android:auth0:1.+'
}
```

If your application does not plan to make use of the Web Authentication module provided by the SDK, you will need to remove the unused activity from the `AndroidManifest.xml` file to prevent Manifest Placeholder issues. That can be achieved by adding an activity declaration and annotating it with `tools:node="remove"`.

```xml
<application>
  <!-- Add the activity declaration line below -->
   <activity
    android:name="com.auth0.android.provider.AuthenticationActivity"
    tools:node="remove" />
  
</application>
```

However, if you do plan to support Web Authentication, head over [here](https://auth0.com/docs/libraries/auth0-android#authentication-via-universal-login) to learn how to declare the Manifest Placeholders.

Now is time to run the Gradle Sync task in order to refresh the project and its dependencies.