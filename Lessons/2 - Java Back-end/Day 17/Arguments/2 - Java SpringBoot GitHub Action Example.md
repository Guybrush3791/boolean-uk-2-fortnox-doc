# Java SpringBoot GitHub Action Example
## Create project and push to repository
### Create new project from [start.spring.io](https://start.spring.io/)
Create a new project through [start.spring.io](https://start.spring.io/) UI or via `cli` and give it proper name
```sh file:"New SpringBoot project download command" hlt:|$PROJECTNAME
curl https://start.spring.io/starter.zip               \
  -d type=gradle-project                               \
  -d language=java                                     \
  -d bootVersion=3.5.5                                 \
  -d baseDir=$PROJECTNAME                              \
  -d groupId=org.booleanuk                             \
  -d artifactId=$PROJECTNAME                           \
  -d name=$PROJECTNAME                                 \
  -d description="Demo project for Spring Boot"        \
  -d packageName=org.booleanuk.demo                    \
  -d packaging=jar                                     \
  -d javaVersion=21                                    \
  -d dependencies=web,devtools                         \
  -o $PROJECTNAME.zip
```

### Add automation `yaml` file
Extract the archive and open project in *IntelliJ*. Once project is loaded, add a simple `pr-validation.yaml` file to the `.github/workflows` folder
```yaml file:pr-validation.yaml
name: Simple CI/CD Pipeline # [!note] Defines the workflow name that appears in GitHub Actions tab

on:
  push:
    branches: [ main ] # [!note] Triggered on `push` on `main` branch
  pull_request:
    branches: [ main ] # [!note] Triggered on `pool request` on `main` branch

jobs: 
  build-and-deploy:
    runs-on: ubuntu-latest # [!note] Specifies the runner environment (Ubuntu virtual machine)
    
    steps: 
      - name: Checkout Code 
        uses: actions/checkout@v4 # [!note] Uses `pre-built action` to `download repository code` to runner
        
      - name: Setup Java 21 
        uses: actions/setup-java@v4 # [!note] Uses `pre-built action` to `install and configure Java`
        with: 
          java-version: '21' # [!note] Specifies Java version 21 to install
          distribution: 'temurin' # [!note] Specifies Eclipse Temurin as the Java distribution
          
      - name: Run Tests 
        run: ./gradlew test # [!note] Executes Gradle test command `to run unit/integration tests`
        
      - name: Build Application 
        run: ./gradlew bootJar # [!note] Executes Gradle command to `create Spring Boot JAR file`
        
      - name: Create Release 
        if: github.ref == 'refs/heads/main' # [!note] `Conditional execution` - only runs on main branch pushes (not PRs)
        run: | # [!note] Multi-line shell script block to `deploy project`
          VERSION="1.0.${{ github.run_number }}"
          echo "Built version: $VERSION" 
          echo "Ready for deployment!" 

```

### Push project to `main` branch
Create a new *GitHub repository* and init with the actual project
```sh file:"Init new repository"
git init
git add -A
git commit -m "init"
git branch -M main
git remote add origin ${your-repo}
git push -u origin main
```

## Create new branch and deploy feature
### Create branch from `main` 
Now you are on *updated main branch*, it's time to create a new *feature*
```prompt:fish file:"Checkout new branch"
git checkout -b feature/health-check
```

### Develop feature
Once moved to the new *branch* you can develop *test* and *actual implementation* classes 
```java file:HealthControllerTest.java group:health-check
@WebMvcTest(HealthController.class)  // Only loads web layer  
class HealthControllerTest {  
  
    private HealthController healthController;  
  
    public HealthControllerTest() {  
  
        healthController = new HealthController();  
    }  
  
    @Test  
    void healthEndpointReturnsOK() throws Exception {  
  
        assertEquals("OK", healthController.health());  
    }  
}
```
```java file:HealthController.java group:health-check
@RestController  
@RequestMapping("/api/health")  
public class HealthController {  
  
    @GetMapping  
    public String health() {  
  
        return "OK";  
    }  
}
```
> [!note] Make sure your test is *actually passing* through `{sh} ./gradlew test`

### Update new branch
Now you can push updates to the repository through some UI or via *cli*
```sh file:"Commit & push new feature"
git add -A
git commit -m "feat: add health check endpoint with tests

- Add GET /api/health endpoint returning OK status
- Add integration test for health endpoint
- Ensures service liveness monitoring capability"
  
git push origin feature/health-check
```

## Pool Request and deploy
### Create a new `PR`
To create new `PR` as always and check test execution. If all tests pass, you can then merge to the `main` *branch*. Whole test suit will be executed, but this time last *stage* will be executed, too. The project is finally deployed to production stage.

![[GitHub Automation CI-CD push & deploy|1000]]

---

![[Lessons/2 - Java Back-end/Day 17/__block/Links]]