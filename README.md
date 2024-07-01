This is a small repro of issue: https://youtrack.jetbrains.com/issue/KT-69330/KotlinCompile-friendPathsSet-property-is-racy-due-causing-build-cache-invalidation


The steps below show how we did this.

1. Download from this wizard: https://kmp.jetbrains.com/
2. Open in Intellij

3. Open `composeApp/desktopMain/kotlin/main.kt` and make it look like the following (just added a function):
    ```
    import androidx.compose.ui.window.Window
    import androidx.compose.ui.window.application
    
    fun main() = application {
        Window(
            onCloseRequest = ::exitApplication,
            title = "KotlinProject",
        ) {
            App()
            alwaysReturnTrue()
        }
    }
    
    fun alwaysReturnTrue(): Boolean = true
    ```

4. Go to composeApp/build.gradle.kts and add under the sourceSets dsl:
    ```
    val desktopTest by getting
    desktopTest.dependencies {
        implementation(libs.junit)
        implementation(libs.kotlin.test)
    }
    ```

5. Create the desktopTest/kotlin dir under composeApp (I did via UI) and create a test file, adding this code:
    ```
    import org.junit.Test
    import kotlin.test.assertTrue
    
    class MyTest {
    
        @Test
        fun checkTest() {
            assertTrue( alwaysReturnTrue() )
        }
    
    }
    ```

6. In the command line, run:
    ```
    ./gradlew :composeApp:compileTestKotlinDesktop --no-configuration-cache 
    ```
    twice, the second time, it will say `8 actionable tasks: 8 up-to-date`


7. Then run:
    ```
    ./gradlew :composeApp:desktopJar --no-configuration-cache 
    ```

8. Then run:
    ```
    ./gradlew :composeApp:compileTestKotlinDesktop --no-configuration-cache  --info
    ```

    again and you will notice it says: `8 actionable tasks: 1 executed, 7 up-to-date`, and in the printed logs:
   
   ```
    Task ':composeApp:compileTestKotlinDesktop' is not up-to-date because:
    Value of input property 'friendPathsSet$kotlin_gradle_plugin_common' has changed for task ':composeApp:compileTestKotlinDesktop'
   ```

   Example build scan can be found here: https://scans.gradle.com/s/wlkibikocxlkc/timeline?details=l3lv65nzzvhvo