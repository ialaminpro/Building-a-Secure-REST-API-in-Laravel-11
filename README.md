Here’s a comprehensive guide to creating a RESTful API CRUD application in Laravel 11 with a focus on managing courses instead of products. This approach incorporates best practices, including using Laravel Passport for API authentication.

## Title: **Building a RESTful API in Laravel 11 with Passport Authentication**

### Step 1: Setting Up Laravel

Start by creating a new Laravel project using Composer:

```bash
composer create-project --prefer-dist laravel/laravel rest-api-course
```

### Step 2: MySQL Database Configuration

By default, Laravel uses SQLite for database connections. To switch to MySQL, open the `.env` file and change the following line:

```plaintext
DB_CONNECTION=mysql
```

Make sure to set the correct database credentials.

### Step 3: Create the Course Model with Migration

Generate the Course model along with a migration file:

```bash
php artisan make:model Course -a
```

### Step 4: Migration

Open the migration file located in `database/migrations/YYYY_MM_DD_HHMMSS_create_courses_table.php` and modify the `up` method:

```php
public function up(): void
{
    Schema::create('courses', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('description');
        $table->timestamps();
    });
}
```

Run the migration to create the `courses` table:

```bash
php artisan migrate
```

### Step 5: Create Course Repository Interface

Create a repository interface for the Course model:

```bash
php artisan make:interface Interfaces/CourseRepositoryInterface
```

In the `Interfaces` directory, create a file called `CourseRepositoryInterface.php` and add the following code:

```php
namespace App\Interfaces;

interface CourseRepositoryInterface
{
    public function index();
    public function getById($id);
    public function store(array $data);
    public function update(array $data, $id);
    public function delete($id);
}
```

### Step 6: Create Course Repository Class

Next, create a repository class for the Course model:

```bash
php artisan make:class Repositories/CourseRepository
```

In the `Repositories` directory, create a file called `CourseRepository.php` and implement the following code:

```php
namespace App\Repository;

use App\Models\Course;
use App\Interfaces\CourseRepositoryInterface;

class CourseRepository implements CourseRepositoryInterface
{
    public function index()
    {
        return Course::all();
    }

    public function getById($id)
    {
        return Course::findOrFail($id);
    }

    public function store(array $data)
    {
        return Course::create($data);
    }

    public function update(array $data, $id)
    {
        return Course::whereId($id)->update($data);
    }

    public function delete($id)
    {
        Course::destroy($id);
    }
}
```

### Step 7: Bind the Interface to the Implementation

To bind the `CourseRepository` to the `CourseRepositoryInterface`, create a service provider:

```bash
php artisan make:provider RepositoryServiceProvider
```

Open `app/Providers/RepositoryServiceProvider.php` and update the `register` method:

```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Interfaces\CourseRepositoryInterface;
use App\Repository\CourseRepository;

class RepositoryServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(CourseRepositoryInterface::class, CourseRepository::class);
    }

    public function boot(): void
    {
        //
    }
}
```

### Step 8: Request Validation

Create two request classes: `StoreCourseRequest` and `UpdateCourseRequest`.

Generate the classes using:

```bash
php artisan make:request StoreCourseRequest
php artisan make:request UpdateCourseRequest
```

In `StoreCourseRequest`, add the following code:

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Http\Exceptions\HttpResponseException;
use Illuminate\Contracts\Validation\Validator;

class StoreCourseRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => 'required',
            'description' => 'required'
        ];
    }

    public function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(response()->json([
            'success' => false,
            'message' => 'Validation errors',
            'data' => $validator->errors()
        ]));
    }
}
```

In `UpdateCourseRequest`, add similar validation rules:

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Http\Exceptions\HttpResponseException;
use Illuminate\Contracts\Validation\Validator;

class UpdateCourseRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => 'required',
            'description' => 'required'
        ];
    }

    public function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(response()->json([
            'success' => false,
            'message' => 'Validation errors',
            'data' => $validator->errors()
        ]));
    }
}
```

### Step 8: Structuring API Responses
To ensure consistency in API responses, we can create a utility response class similar to the one discussed earlier. This allows us to standardize responses throughout the application.

Create a class `ApiResponse.php` in the `app/Http/Helpers` directory:
```bash
namespace App\Http\Helpers;

class ApiResponse
{
    public static function success($data, $status = 200)
    {
        return response()->json(['status' => 'success', 'data' => $data], $status);
    }

    public static function error($message, $status)
    {
        return response()->json(['status' => 'error', 'message' => $message], $status);
    }
}
```
Now, update the controller methods to use this `ApiResponse` class for standardized responses.

### Step 8: Testing the API
You can test your API endpoints using tools like Postman or cURL. Here's how you would test the registration endpoint:
```bash
curl --request POST \
  --url http://localhost:8000/api/register \
  --header 'Content-Type: application/json' \
  --data '{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "password",
    "password_confirmation": "password"
}'
```

To log in and retrieve an access token, you can use the login endpoint:
```bash
curl --request POST \
  --url http://localhost:8000/api/login \
  --header 'Content-Type: application/json' \
  --data '{
    "email": "john@example.com",
    "password": "password"
}'
```


### Step 9: Create a Common API Response Class

Create a response class to standardize API responses:


```bash
php artisan make:class Classes/ApiResponse
```

Add the following code to `ApiResponse.php`:

```php
namespace App\Classes;

use Illuminate\Support\Facades\DB;
use Illuminate\Http\Exceptions\HttpResponseException;
use Illuminate\Support\Facades\Log;

class ApiResponse
{
    public static function rollback($e, $message = "Something went wrong! Process not completed")
    {
        DB::rollBack();
        self::throw($e, $message);
    }

    public static function throw($e, $message = "Something went wrong! Process not completed")
    {
        Log::info($e);
        throw new HttpResponseException(response()->json(["message" => $message], 500));
    }

    public static function sendResponse($result, $message, $code = 200)
    {
        $response = [
            'success' => true,
            'data' => $result
        ];
        if (!empty($message)) {
            $response['message'] = $message;
        }
        return response()->json($response, $code);
    }
}
```

### Step 10: Create Course Resource

Generate a resource for the Course model:

```bash
php artisan make:resource CourseResource
```

In `CourseResource.php`, transform the resource:

```php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class CourseResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'description' => $this->description
        ];
    }
}
```

### Step 11: Create the CourseController Class

Open `app/Http/Controllers/CourseController.php` and implement the CRUD operations:

```php
namespace App\Http\Controllers;

use App\Http\Requests\StoreCourseRequest;
use App\Http\Requests\UpdateCourseRequest;
use App\Interfaces\CourseRepositoryInterface;
use App\Classes\ApiResponse;
use App\Http\Resources\CourseResource;
use Illuminate\Support\Facades\DB;

class CourseController extends Controller
{
    private CourseRepositoryInterface $courseRepositoryInterface;

    public function __construct(CourseRepositoryInterface $courseRepositoryInterface)
    {
        $this->courseRepositoryInterface = $courseRepositoryInterface;
    }

    public function index()
    {
        $data = $this->courseRepositoryInterface->index();
        return ApiResponse::sendResponse(CourseResource::collection($data), '', 200);
    }

    public function store(StoreCourseRequest $request)
    {
        $details = [
            'title' => $request->title,
            'description' => $request->description
        ];

        DB::beginTransaction();
        try {
            $course = $this->courseRepositoryInterface->store($details);
            DB::commit();
            return ApiResponse::sendResponse(new CourseResource($course), 'Course created successfully', 201);
        } catch (\Exception $ex) {
            return ApiResponse::rollback($ex);
        }
    }

    public function show($id)
    {
        $course = $this->courseRepositoryInterface->getById($id);
        return ApiResponse::sendResponse(new CourseResource($course), '', 200);
    }

    public function update(UpdateCourseRequest $request, $id)
    {
        $updateDetails = [
            'title' => $request->title,
            'description' => $request->description
        ];

        DB::beginTransaction();
        try {
            $this->courseRepositoryInterface->update($updateDetails, $id);
            DB::commit();
            return ApiResponse::sendResponse(null, 'Course updated successfully', 204);
        } catch (\Exception $ex) {
            return ApiResponse::rollback($ex);
        }
    }

    public function destroy($id)
    {
        $this->courseRepositoryInterface->delete($id);
        return ApiResponse::sendResponse(null, 'Course deleted successfully', 204);
    }
}
```

### Step 12: Set Up API Routes

Open `routes/api.php` and add the following code to map the API routes:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\CourseController;

Route::middleware('auth:api')->group(function () {
    Route::

apiResource('/courses', CourseController::class);
});
```

### Step 13: Setting Up Laravel Passport

Install Laravel Passport:

```bash
composer require laravel/passport
```

Run the migrations:

```bash
php artisan migrate
```

Install Passport:

```bash
php artisan passport:install
```

Executing the subsequent command allows you to publish the API route file:

```bash
php artisan install:api -- passport
```

In `config/auth.php`, set the `api` guard to use Passport:

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

### Step 14: Protecting Routes with Passport

In the `User` model (`app/Models/User.php`), add the `HasApiTokens` trait:

```php
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;
    // ...
}
```

### Step 15: Testing the API

You can now run your Laravel application:

```bash
php artisan serve
```

### Conclusion

You have successfully set up a RESTful API for managing courses in Laravel 11, implementing CRUD operations with best practices, including validation, repository design pattern, and using Passport for API authentication. This structured approach not only promotes clean code but also makes it easier to manage and scale your application in the future.
