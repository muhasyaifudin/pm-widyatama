## Blog App

### **`database/migrations`**

**database/migrations/*****_create_posts_table.php**
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->foreignId('category_id');
            $table->string('slug')->unique();
            $table->string('title');
            $table->text('body');
            $table->text('excerpt');
            $table->string('thumbnail')->nullable();
            $table->timestamps();
            $table->timestamp('published_at')->nullable();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

**database/migrations/*****_create_categories_table.php**
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string('name')->unique();
            $table->string('slug')->unique();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('categories');
    }
};
```

**database/migrations/*****_create_comments_table.php**
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('comments', function (Blueprint $table) {
            $table->id();
            $table->foreignId('post_id')->constrained()->cascadeOnDelete();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->text('body');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('comments');
    }
};
```

**database/seeders/DatabaseSeeder.php**
```php
<?php

namespace Database\Seeders;


use App\Models\Category;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
       

        $personal = Category::create([
            'name' => 'Personal',
            'slug' => 'personal'
        ]);
        $family = Category::create([
            'name' => 'Family',
            'slug' => 'family'
        ]);
        $work = Category::create([
            'name' => 'Work',
            'slug' => 'work'
        ]);

        
    }
}
```

##

### **`app/Models`**

**app/Models/Category.php**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Category extends Model
{
    use HasFactory;

    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

**app/Models/Comment.php**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    use HasFactory;

    public function post()
    {
        return $this->belongsTo(Post::class);

    }

    public function author()
    {
        return $this->belongsTo(User::class, 'user_id');
    }
}
```

**app/Models/Post.php**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    use HasFactory;

    protected $guarded = ['id'];
    protected $with = ['category', 'author'];

    public function scopeFilter($query, array $filters): void
    {
        $query->when($filters['search'] ?? false, fn($query, $search) => $query->where(fn($query) => $query
            ->where('title', 'like', '%' . $search . '%')
            ->orWhere('body', 'like', '%' . $search . '%')));

        $query->when($filters['category'] ?? false, fn($query, $category) => $query
            ->whereHas('category', fn($query) => $query->where('slug', $category))
        );

        $query->when($filters['author'] ?? false, fn($query, $author) => $query
            ->whereHas('author', fn($query) => $query->where('username', $author))
        );

    }


    public function category()
    {
        return $this->belongsTo(Category::class);
    }

    public function author(): BelongsTo
    {
        return $this->belongsTo(User::class, 'user_id');
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
```

**app/Models/User.php**
```php
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $guarded = [];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var array<int, string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];

    public function setPasswordAttribute($password): void
    {
        $this->attributes['password'] = bcrypt($password);
    }

    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

##

### **`app/Http/Controllers`**

**app/Http/Controllers/AdminPostController.php**
```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;
use Illuminate\Validation\Rule;

class AdminPostController extends Controller
{
    public function index()
    {
        return view('admin.posts.index', [
            'posts' => Post::paginate(50)
        ]);
    }

    public function edit(Post $post)
    {
        return view('admin.posts.edit', ['posts' => $post]);
    }

    public function update(Post $post)
    {
        $attribute = $this->validatePost($post);

        if ($attribute['thumbnail'] ?? false) {
            $attribute['thumbnail'] = request()->file('thumbnail')->store('thumbnail');
        }

        $post->update($attribute);
        return back()->with('success', 'The Post Has Been Updated!');
    }

    protected function validatePost(?Post $post = null): array
    {
        $post ??= new Post();

        return request()->validate([
            'title' => 'required',
            'slug' => ['required', Rule::unique('posts', 'slug')->ignore($post)],
            'excerpt' => 'required',
            'body' => 'required',
            'thumbnail' => $post->exists() ? ['image'] : ['required', 'image'],
            'category_id' => ['required', Rule::exists('categories', 'id')]
        ]);
    }

    public function store()
    {
        $attribute = $this->validatePost(new Post());
        $attribute['user_id'] = auth()->id();
        $attribute['thumbnail'] = request()->file('thumbnail')->store('thumbnails');

        Post::create($attribute);

        return redirect('/');
    }

    public function create(Post $post)
    {
        return view('admin.posts.create');
    }

    public function destory(Post $post)
    {
        $post->delete();

        return back()->with('success', 'The Post Has Been Deleted!');

    }
}
```

**app/Http/Controllers/PostCommentsController.php**
```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostCommentsController extends Controller
{
    public function store(Post $post): \Illuminate\Http\RedirectResponse
    {
        request()->validate([
            'body' => 'required'
        ]);

        $post->comments()->create([
            'user_id' => request()->user()->id,
            'body' => request('body')
        ]);
        return back();
    }
}
```


**app/Http/Controllers/PostController.php**
```php
<?php

namespace App\Http\Controllers;

use App\Models\Category;
use App\Models\Post;
use Illuminate\Http\Request;
use Illuminate\Validation\Rule;

class PostController extends Controller
{
    public function index()
    {
        return view('posts.index', [
            'posts' => Post::latest()->filter(request(['search', 'category', 'author']))->paginate(6)->withQueryString(),
            'currentCategory' => Category::firstWhere('slug', request('category'))
        ]);
    }

    public function show(Post $post)
    {
        return view('posts.show', [
            'post' => $post
        ]);
    }
}
```

**app/Http/Controllers/RegisterController.php**
```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class RegisterController extends Controller
{
    public function store()
    {

        $attributes = request()->validate([
            'name' => ['required', 'max:255'],
            'username' => ['required', 'max:255', 'min:3', 'unique:users,username'],
            'email' => ['required', 'email', 'max:255', 'unique:users,email'],
            'password' => ['required', 'max:255', 'min:7']
        ]);

        $user = User::create($attributes);
        
        auth()->login($user);

        return redirect('/')->with('success', 'Your Account has been created.');
    }

    public function create()
    {
        return view('register.create');
    }


}
```


**app/Http/Controllers/SessionsController.php**
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Validation\ValidationException;

class SessionsController extends Controller
{
    public function create()
    {
        return view('sessions.create');
    }

    public function store()
    {
        $attributes = request()->validate([
            'email' => ['required', 'email'],
            'password' => 'required'
        ]);

        if (auth()->attempt($attributes)) {
            throw ValidationException::withMessages(['email' => 'Your Provided Credentials Could Not Be Verified']);
        }

        session()->regenerate();

        return redirect('/')->with('success', 'Welcome Back!');
    }

    public function destroy()
    {
        auth()->logout();

        return redirect('/')->with('success', 'Goodbye!');
    }
}
```

##

### **`routes`**

**routes/web.php**
```php
<?php /** @noinspection ALL */

use App\Http\Controllers\AdminPostController;
use App\Http\Controllers\NewsletterController;
use App\Http\Controllers\PostCommentsController;
use App\Http\Controllers\SessionsController;
use App\Models\Post;
use App\Models\User;
use App\Models\Category;
use App\Services\Newsletter;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\PostController;
use App\Http\Controllers\RegisterController;


Route::get('/', [PostController::class, 'index'])->name('home');

Route::get('posts/{post:slug}', [PostController::class, 'show']);
Route::post('posts/{post:slug}/comments', [PostCommentsController::class, 'store']);

Route::post('newsletter', NewsletterController::class);

Route::get('register', [RegisterController::class, 'create'])->middleware('guest');
Route::post('register', [RegisterController::class, 'store'])->middleware('guest');

Route::get('login', [SessionsController::class, 'create'])->middleware('guest');
Route::post('login', [SessionsController::class, 'store'])->middleware('guest');

Route::post('logout', [SessionsController::class, 'destroy'])->middleware('auth');



    Route::get('/admin/posts', [AdminPostController::class, 'index']);
    Route::get('/admin/posts/{post}/edit', [AdminPostController::class, 'edit']);
    Route::get('/admin/posts/create/', [AdminPostController::class, 'create']);
    Route::post('/admin/posts', [AdminPostController::class, 'store']);
    Route::patch('/admin/posts/{post}', [AdminPostController::class, 'update']);
    Route::delete('/admin/posts/{post}', [AdminPostController::class, 'destory']);


```

##

### **`app/View/Components`**

**app/View/Components/CategoryDropdown.php**
```php
<?php

namespace App\View\Components;

use App\Models\Category;
use Closure;
use Illuminate\Contracts\View\View;
use Illuminate\View\Component;

class CategoryDropdown extends Component
{

    /**
     * Get the view / contents that represent the component.
     */
    public function render(): View|Closure|string
    {
        return view('components.category-dropdown', ['categories' => Category::all(),
            'currentCategory' => Category::firstWhere('slug', request('category'))]);
    }
}
```

##


### **`resources/views`**

**resources/views/components/form/button.blade.php**
```html
<button type="submit"
        class="bg-blue-500 text-white uppercase font-semibold text-xs py-2 px-10 rounded-2xl hover:bg-blue-600 mt-5">
    {{ $slot }}
</button>

```

**resources/views/components/form/error.blade.php**
```html
@props(['name'])

@error($name)
<p class="text-red-500 text-xs mt-2">{{ $message }}</p>
@enderror
```

**resources/views/components/form/field.blade.php**
```html
<div class="mt-6">
    {{ $slot }}
</div>
```

**resources/views/components/form/form-input.blade.php**
```html
@props(['name'])
<x-form.field>
    <x-form.label name="{{ $name }}"/>
    <input name="{{ $name }}" id="{{ $name }}"
           {{ $attributes(['value' => old($name)]) }}
           class="border border-gray-200 p-2 w-full rounded">
    <x-form.error name="{{ $name }}"/>
</x-form.field>
```

**resources/views/components/form/label.blade.php**
```html
@props(['name'])

<label for="{{ $name }}" class="block mb-2 uppercase font-bold text-xs text-gray-700">{{ ucwords($name) }}</label>
```

**resources/views/components/form/textarea.blade.php**
```html
@props(['name'])
<x-form.field>
    <x-form.label name="{{ $name }}"/>
    <textarea class="border border-gray-200 rounded p-2 w-full" name="{{ $name }}" id="{{ $name }}"
              required
            {{ $attributes  }}> {{ $slot ?? old($name)}}
    </textarea>

    <x-form.error name="{{ $name }}"/>
</x-form.field>

```

**resources/views/components/category-dropdown.blade.php**
```html
<x-dropdown id="dropdown">
    <x-slot name="trigger">
        <button class="py-2 pl-3 pr-9 text-sm w-full font-semibold lg:w-32 text-left lg:inline-flex">
            {{ isset($currentCategory) ? ucwords($currentCategory->name) : 'Categories' }}
            <x-icon name="down-arrow" class="absolute pointer-events-none" style="right: 12px;"/>
        </button>

    </x-slot>
    <x-dropdown-item href="/?{{ http_build_query(request()->except('category', 'page')) }}"
                     :active="request()->routeIs('home')">
        All
    </x-dropdown-item>

    @foreach ($categories as $category)
        <x-dropdown-item
            href="/?category={{ $category->slug }}&{{ http_build_query(request()->except('category', 'page')) }}"
            :active='request()->is("categories/$category->slug")'>
            {{ ucwords($category->name) }}</x-dropdown-item>
    @endforeach
</x-dropdown>
```

**resources/views/components/dropdown-item.blade.php**
```html
@props(['active' => false])

@php
    $classes = 'block text-left px-3 text-sm leading-6 focus:bg-blue-300 hover:bg-blue-500 hover:text-white focus:text-white';

    if ($active) {
        $classes .= ' bg-blue-500 text-white ';
    }
@endphp

<a {{ $attributes(['class' => $classes]) }}>
    {{ $slot }}
</a>
```

**resources/views/components/dropdown.blade.php**
```html
@props(['trigger'])

<div x-data="{ show: false }" @click.away="show = false" class="relative">
    {{-- trigger --}}
    <div @click="show = ! show">
        {{ $trigger }}
    </div>

    {{-- Links --}}
    <div x-show="show" class="py-2 absolute bg-gray-100 w-full z-50 mt-2 rounded-xl overflow-auto max-h-52"
         style="display: none">
        {{ $slot }}
    </div>
</div>
```

**resources/views/components/flash.blade.php**
```html
@if(session()->has('success'))
    <div x-data="{ show: true }"
         x-init="setTimeout(() => show = false, 4000)"
         x-show="show"
         class="fixed right-0 bg-blue-500 text-white py-2 px-4 rounded-xl bottom-3 right-3 text-sm">
        <p>
            {{ session('success') }}
        </p>
    </div>
@endif
```

**resources/views/components/icon.blade.php**
```html
@props(['name'])

@if ($name === 'down-arrow')
    <svg {{ $attributes(['class' => 'transform -rotate-90']) }} width="22" height="22" viewBox="0 0 22 22">
        <g fill="none" fill-rule="evenodd">
            <path stroke="#000" stroke-opacity=".012" stroke-width=".5" d="M21 1v20.16H.84V1z">
            </path>
            <path fill="#222" d="M13.854 7.224l-3.847 3.856 3.847 3.856-1.184 1.184-5.04-5.04 5.04-5.04z">
            </path>
        </g>
    </svg>
@endif
```

**resources/views/components/layout.blade.php**
```html
<!doctype html>

<title>My Blog</title>
<link href="https://unpkg.com/tailwindcss@^2/dist/tailwind.min.css" rel="stylesheet">
<link rel="preconnect" href="https://fonts.gstatic.com">
<link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;600;700&display=swap" rel="stylesheet">
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
<style>
    html {
        scroll-behavior: smooth;
    }
</style>
<body style="font-family: Open Sans, sans-serif">
<section class="px-6 py-8">
    <nav class="md:flex md:justify-between md:items-center">
        <div>
            <a href="/">
                <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/36/Logo.min.svg/352px-Logo.min.svg.png?20200603074624" alt="Laravel Logo" width="165" height="16">
            </a>
        </div>

        <div class="mt-8 md:mt-0 flex items-center">

            @auth()
                <x-dropdown>
                    <x-slot name="trigger">
                        <button class="text-xs font-bold uppercase">Welcome, {{ auth()->user()->name }}!</button>
                    </x-slot>

                        <x-dropdown-item href="/admin/posts/" :active="request()->is('admin/posts')">
                            Dashboard
                        </x-dropdown-item>
                        <x-dropdown-item href="/admin/posts/create/" :active="request()->is('admin/posts/create')">
                            New Post
                        </x-dropdown-item>


                    <x-dropdown-item href="#" x-data="{}"
                                     @click.prevent="document.querySelector('#logout-form').submit()">
                        Log out
                    </x-dropdown-item>
                    <form action="/logout" method="post" id="logout-form" class="hidden">
                        @csrf
                    </form>

                </x-dropdown>

            @else
                <a href="/register" class="text-xs font-bold uppercase">Register</a>
                <a href="/login" class="ml-3 text-xs font-bold uppercase">Log in</a>
            @endauth
            <a href="#newsletter"
               class="bg-blue-500 ml-3 rounded-full text-xs font-semibold text-white uppercase py-3 px-5">
                Subscribe for Updates
            </a>
        </div>
    </nav>

    {{ $slot }}

    <footer id="newsletter"
            class="bg-gray-100 border border-black border-opacity-5 rounded-xl text-center py-16 px-10 mt-16">
        <img src="/images/lary-newsletter-icon.svg" alt="" class="mx-auto -mb-6" style="width: 145px;">
        <h5 class="text-3xl">Stay in touch with the latest posts</h5>
        <p class="text-sm mt-3">Promise to keep the inbox clean. No bugs.</p>

        <div class="mt-10">
            <div class="relative inline-block mx-auto lg:bg-gray-200 rounded-full">

                <form method="POST" action="/newsletter" class="lg:flex text-sm">
                    @csrf

                    <div class="lg:py-3 lg:px-5 flex items-center">
                        <label for="email" class="hidden lg:inline-block">
                            <img src="/images/mailbox-icon.svg" alt="mailbox letter">
                        </label>

                        <input name="email" id="email" type="text" placeholder="Your email address"
                               class="lg:bg-transparent py-2 lg:py-0 pl-4 focus-within:outline-none">
                        @error('email')
                        <span class="text-xs text-red-500"> {{ $message }}</span>
                        @enderror

                    </div>

                    <button type="submit"
                            class="transition-colors duration-300 bg-blue-500 hover:bg-blue-600 mt-4 lg:mt-0 lg:ml-3 rounded-full text-xs font-semibold text-white uppercase py-3 px-8">
                        Subscribe
                    </button>
                </form>
            </div>
        </div>
    </footer>
</section>
<x-flash/>
</body>
```

**resources/views/components/panel.blade.php**
```html
<div {{ $attributes(['class' => 'border border-gray-200 p-6 rounded-xl']) }}>
    {{ $slot }}
</div>
```

**resources/views/components/post-card.blade.php**
```html
@props(['post'])

<article {{ $attributes->merge(['class' => 'transition-colors duration-300 hover:bg-gray-100 border border-black border-opacity-0 hover:border-opacity-5 rounded-xl'])}}>
    <div class="py-6 px-5">
        <div>
            <img src="{{ asset('storage/' . $post->thumbnail) }}" alt="Blog Post illustration" class="rounded-xl">
        </div>

        <div class="mt-8 flex flex-col justify-between">
            <header>
                <div class="space-x-2">
                    <a href="?category={{ $post->category->slug }}"
                       class="px-3 py-1 border border-blue-300 rounded-full text-blue-300 text-xs uppercase font-semibold"
                       style="font-size: 10px">{{ $post->category->name }}</a>
                </div>

                <div class="mt-4">
                    <h1 class="text-3xl">
                        <a href="/posts/{{ $post->slug }}"></a>
                        {{ $post->title }}
                    </h1>

                    <span class="mt-2 block text-gray-400 text-xs">
                        Published <time>{{ $post->created_at->diffForHumans() }}</time>
                    </span>

                </div>
            </header>

            <div class="text-sm mt-4 space-y-4">
                {!! $post->excerpt !!}
            </div>

            <footer class="flex justify-between items-center mt-8">
                <div class="flex items-center text-sm">
                    <img src="/images/lary-avatar.svg" alt="Lary avatar">
                    <div class="ml-3">
                        <h5 class="font-bold">
                            <a href="/?author={{ $post->author->username }}">{{ $post->author->name }}</a></h5>

                    </div>
                </div>

                <div>
                    <a href="/posts/{{ $post->slug }}"
                       class="transition-colors duration-300 text-xs font-semibold bg-gray-200 hover:bg-gray-300 rounded-full py-2 px-8">Read
                        More</a>
                </div>
            </footer>
        </div>
    </div>
</article>
```

**resources/views/components/post-comment.blade.php**
```html
@props(['comment'])
<x-panel class="bg-gray-50">
    <article class="flex space-x-4">
        <div class="flex-shrink-0">
            <img src="https://i.pravatar.cc/60?u={{ $comment->user_id }}" alt="" width="80" height="80"
                 class="rounded-xl">
        </div>
        <div>
            <header class="mb-4">
                <h3 class="font-bold">{{ $comment->author->username }}</h3>

                <p class="text-xs"> Posted on
                    <time> {{ $comment->created_at->format('F j, Y, g:i a') }}</time>
                </p>
            </header>
            <p>
                {{ $comment->body }}
            </p>
        </div>
    </article>
</x-panel>
```

**resources/views/components/post-featured-card.blade.php**
```html
@props(['post'])

<article
    class="transition-colors duration-300 hover:bg-gray-100 border border-black border-opacity-0 hover:border-opacity-5 rounded-xl">
    <div class="py-6 px-5 lg:flex">
        <div class="flex-1 lg:mr-8">
            <img src="{{ asset('storage/' . $post->thumbnail) }}" alt="Blog Post illustration" class="rounded-xl">
        </div>

        <div class="flex-1 flex flex-col justify-between">
            <header class="mt-8 lg:mt-0">
                <div class="space-x-2">
                    <a href="/categories/{{ $post->category->slug }}"
                       class="px-3 py-1 border border-blue-300 rounded-full text-blue-300 text-xs uppercase font-semibold"
                       style="font-size: 10px">{{ $post->category->name }}</a>
                </div>

                <div class="mt-4">
                    <h1 class="text-3xl">
                        <a href="/posts/{{ $post->slug }}"></a>
                        {{ $post->title }}
                    </h1>

                    <span class="mt-2 block text-gray-400 text-xs">
                        Published <time>{{ $post->created_at->diffForHumans() }}</time>
                    </span>
                </div>
            </header>

            <div class="text-sm mt-4 space-y-4">
                {!! $post->excerpt !!}
            </div>

            <footer class="flex justify-between items-center mt-8">
                <div class="flex items-center text-sm">
                    <img src="/images/lary-avatar.svg" alt="Lary avatar">
                    <div class="ml-3">
                        <h5 class="font-bold">
                            <a href="/?author={{ $post->author->username }}">{{ $post->author->name }}</a>
                        </h5>
                    </div>
                </div>

                <div class="hidden lg:block">
                    <a href="/posts/{{ $post->slug }}"
                       class="transition-colors duration-300 text-xs font-semibold bg-gray-200 hover:bg-gray-300 rounded-full py-2 px-8">Read
                        More</a>
                </div>
            </footer>
        </div>
    </div>
</article>
```

**resources/views/components/posts-cards.blade.php**
```html
@props(['posts'])

<x-post-featured-card :post="$posts[0]"/>

@if ($posts->count() > 1)
    <div class="lg:grid lg:grid-cols-6">
        @foreach ($posts->skip(1) as $post)
            <x-post-card :post="$post" class="{{ $loop->iteration < 3 ? 'col-span-3' : 'col-span-2' }}"/>
        @endforeach
    </div>
@endif
```

**resources/views/components/setting.blade.php**
```html
@props(['heading'])
<section class="py-8 max-w-5xl mx-auto">
    <h1 class="text-lg font-bold mb-8 pb-2 border-b">
        {{ $heading }}
    </h1>

    <div class="flex">
        <aside class="w-48 flex-shrink-0">
            <h4 class="font-semi-bold mb-4">Links</h4>
            <ul>
                <li>
                    <!-- To do: create dashboard panel -->
                    <a href="/admin/posts"
                       class="{{ request()->is('admin/posts') ? 'text-blue-500' : '' }}">All Posts</a>
                    <!------->
                <li>
                    <a href="/admin/posts/create"
                       class="{{ request()->is('admin/posts/create') ? 'text-blue-500' : '' }}">New Post</a>
                </li>
            </ul>
        </aside>
        <main class="flex-1">
            <x-panel>
                {{ $slot }}
            </x-panel>
        </main>
    </div>
</section>
```

**resources/views/components/submit-button.blade.php**
```html
<button type="submit"
        class="bg-blue-500 text-white uppercase font-semibold text-xs py-2 px-10 rounded-2xl hover:bg-blue-600 mt-5">
    {{ $slot }}
</button>
```

**resources/views/posts/_add-comment-from.blade.php**
```html
@auth()
    <x-panel>
        <form action="/posts/{{ $post->slug }}/comments" method="post">
            @csrf

            <header class="flex items-center">
                <img src="https://i.pravatar.cc/60?u={{ auth()->id() }}" alt="" width="80"
                     height="80"
                     class="rounded-full">
                <h2 class="ml-5">
                    Want to share your opinion ?
                </h2>
            </header>

            <div
                class="mt-6">
                <label for="body">
                    <textarea name="body" id="body" rows="6" class="w-full text-sm focus:outline-none focus:ring"
                              placeholder="Write a comment..." required></textarea>
                </label>
                @error('body')
                <span class="text-xs text-red-500">
                                            {{ $message }}
                                        </span>
                @enderror
            </div>

            <div class="flex justify-end mt-6 pt-6 border-t border-gray-200">
                <x-submit-button>
                    Post
                </x-submit-button>
            </div>

        </form>
    </x-panel>
@else
    <p class="font-semibold">
        <a href="/register" class="hover:underline">Register</a> Or
        <a href="/login" class="hover:underline">
            Login to leave a commnet
        </a>
    </p>
@endauth
```

**resources/views/posts/_header.blade.php**
```html
<header class="max-w-xxl mx-auto mt-20 text-center">
    <h1 class="text-4xl">
        The code works  <span class="text-blue-500">on my machine</span> Blog
    </h1>

    <div class="space-y-2 lg:space-y-0 lg:space-x-4 mt-4">
        <!--  Category -->
        <div class="relative lg:inline-flex bg-gray-100 rounded-xl">
            <label for="dropdown" class="sr-only">Categories</label>
            <x-category-dropdown/>

        </div>



        <div class="relative flex lg:inline-flex items-center bg-gray-100 rounded-xl px-3 py-2">
            <label for="search" class="sr-only">Search</label>
            <form method="GET" action="/">
                @if(request('category'))
                    <input type="hidden" name="category" value="{{ request('category') }}">
                @endif

                <input type="text" id="search" placeholder="Find something" name="search"
                       value="{{ request('search') }}"
                       class="bg-transparent placeholder-black font-semibold text-sm">
            </form>
        </div>

    </div>
</header>
```

**resources/views/posts/index.blade.php**
```html
<x-layout>
    @include('posts._header')
    <main class="max-w-6xl mx-auto mt-6 lg:mt-20 space-y-6">
        @if ($posts->count())
            <x-posts-cards :posts="$posts"/>
            {{ $posts->links() }}
        @else
            <p class="text-center">No Post Yet, Come back later</p>
        @endif
    </main>
</x-layout>
```

**resources/views/posts/show.blade.php**
```html
<x-layout>
    <section class="px-6 py-8">
        <main class="max-w-6xl mx-auto mt-6 lg:mt-20 space-y-6">
            <article class="max-w-4xl mx-auto lg:grid lg:grid-cols-12 gap-x-10">
                <div class="col-span-4 lg:text-center lg:pt-14 mb-10">
                    <img src="{{ asset('storage/' . $post->thumbnail) }}" alt="" class="rounded-xl">

                    <p class="mt-4 block text-gray-400 text-xs">
                        Published
                        <time>{{ $post->created_at->diffForHumans() }}</time>
                    </p>

                    <div class="flex items-center lg:justify-center text-sm mt-4">
                        <img src="/images/lary-avatar.svg" alt="Lary avatar">
                        <div class="ml-3 text-left">
                            <p class="font-bold">
                                By <a href="/?author={{ $post->author->username }}">{{ $post->author->name }}</a> in
                                <a href="/categories/{{ $post->category->slug }}">{{ $post->category->name }}</a>
                            </p>
                        </div>
                    </div>
                </div>

                <div class="col-span-8">
                    <div class="hidden lg:flex justify-between mb-6">
                        <a href="/"
                           class="transition-colors duration-300 relative inline-flex items-center text-lg hover:text-blue-500">
                            <svg width="22" height="22" viewBox="0 0 22 22" class="mr-2">
                                <g fill="none" fill-rule="evenodd">
                                    <path stroke="#000" stroke-opacity=".012" stroke-width=".5" d="M21 1v20.16H.84V1z">
                                    </path>
                                    <path class="fill-current"
                                          d="M13.854 7.224l-3.847 3.856 3.847 3.856-1.184 1.184-5.04-5.04 5.04-5.04z">
                                    </path>
                                </g>
                            </svg>
                            Back to Posts
                        </a>
                    </div>

                    <h1 class="font-bold  text-3xl lg:text-4xl mb-10">
                        {{ $post->title }}
                    </h1>

                    <div class="space-y-4 lg:text-lg leading-loose">
                        {!! $post->body !!}
                    </div>
                </div>
                <section class="col-span-8 col-start-5 mt-10 space-y-10">
                    @include('posts._add-comment-from')

                    @foreach ($post->comments as $comment)
                        <x-post-comment :comment="$comment"/>
                    @endforeach

                </section>
            </article>
        </main>
    </section>
</x-layout>
```

**resources/views/register/create.blade.php**
```html
<x-layout>

    <section class="px-6 py-8">
        <main class="max-w-lg mx-auto mt-10 bg-gray-100 border border-gray-300 p-6 rounded-xl">
            <h1 class="text-center font-bold text-xl">Register!</h1>

            <form action="/register" method="post" class="mt-10">
                @csrf

                <div class="mb-6">
                    <label for="name" class="block mb-2 uppercase font-bold text-xs text-gray-700">Name</label>
                    <input type="text" name="name" id="name" value="{{ old('name') }}"
                           class="border border-gray-400 p-2 w-full" required>
                    @error('name')
                    <p class="text-red-500 text-xs mt-1">{{ $message }}</p>
                    @enderror
                </div>

                <div class="mb-6">
                    <label for="email" class="block mb-2 uppercase font-bold text-xs text-gray-700">Email</label>
                    <input type="text" name="email" id="email" value="{{ old('email') }}"
                           class="border border-gray-400 p-2 w-full" required>
                    @error('email')
                    <p class="text-red-500 text-xs mt-1">{{ $message }}</p>
                    @enderror

                </div>

                <div class="mb-6">
                    <label for="username" class="block mb-2 uppercase font-bold text-xs text-gray-700">Username</label>
                    <input type="text" name="username" id="username" value="{{ old('username') }}"
                           class="border border-gray-400 p-2 w-full" required>
                    @error('username')
                    <p class="text-red-500 text-xs mt-1">{{ $message }}</p>
                    @enderror
                </div>

                <div class="mb-6">
                    <label for="password" class="block mb-2 uppercase font-bold text-xs text-gray-700">Password</label>
                    <input type="password" name="password" id="password" class="border border-gray-400 p-2 w-full"
                           required>
                    @error('password')
                    <p class="text-red-500 text-xs mt-1">{{ $message }}</p>
                    @enderror
                </div>

                <div class="mb-6">
                    <button type="submit" class="bg-blue-400 text-white rounded py-2 px-4 hover:bg-blue-500">Submit
                    </button>
                </div>

            </form>

        </main>
    </section>

</x-layout>
```

**resources/views/sessions/create.blade.php**
```html
<x-layout>

    <section class="px-6 py-8">
        <main class="max-w-lg mx-auto mt-10">
            <x-panel>
                <h1 class="text-center font-bold text-xl">Log in!</h1>
                <form action="/login" method="post" class="mt-10">
                    @csrf
                    <x-form.form-input name="email" type="email" autocomplete="username"/>
                    <x-form.form-input name="password" type="password" autocomplete="new-password"/>
                    <x-form.button>
                        Log In
                    </x-form.button>
                </form>
            </x-panel>
        </main>
    </section>
</x-layout>
```




