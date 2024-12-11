props(['active' => false])

@php
	$active
@endphp


:active="request()->is('/')"


{{ attributes }}


{{ slot }}


<x-slot:heading>
	Testing
</x-slot:heading>


props(['active' => false, 'type' => 'a'])

type="button"


<?php if ($type == 'a'):
	//code
?>

<?php else:?>
	//code
?>


@if ($type =='a')
//code
@else
//code
@endif




first()
get()

use($id)

fn($job) => $job['id'] == $id


$job = Arr::first($jobs, fn($job) => $job['id'] == $id);



class Test {
	
	public static function all(): array {
		return [
			['ab'=>2,
			 'asd'=>'wew',
			],
			['ab'=>5,
			 'asd'=>'wzzw',
			]
		]
	}

	public static function find(int $id): array {
		return Arr::first(static:all(), fn($job) => $job['id'} == $id);
	}
}


Test::all();


$test = Test::find($id);


abort(404);




Model naming:

"comments" = Comment
"posts" = Post



$protected table = "job_listings";


public function admin(): static {
	return $this->state(fn (array $attributes) => [
		'admin' => true,
	]);
}

User::factory()->create()
User::factory()->create(100)
User::factory()->unverified()->create()
User::factory()->admin()->create()


$table->id();

$table->unsignedBigInteger('employer_id');
 or $table->foreignIdFor(Employer::class);

'employer_id' => Employer::factory(),



=========DAY 12

// prevent orphan 

$table->foreignIdFor(Employer::class)->constrained()->cascadeOnDelete();
 

php artisan migrate:rollback && php artisan migrate


// turn on foreign key constraints used in sqlite
PRAGMA foreign_keys=on


a.               b.
belongsToMany = belongsToMany



public function tags() {
	return $this->belongsToMany(Tag::class, foreignPivotKey: "job_listing_id");
}

public function jobs() {
	return $this->belongsToMany(Job::class, relatedPivotKey: "job_listing_id");
}


// attach new tag 
$tag->jobs()->attach(Job::find(7));
or $tag->jobs()->attach(7);

		
		get only title. keyword:pluck
		select "column" from "table"
$tag->jobs()->get()->pluck('title')

-------------------



=========DAY 13

https://github.com/barryvdh/laravel-debugbar
composer require barryvdh/laravel-debugbar --dev

APP_DEBUG = true (in .env file)

-----
$jobs = Job::with('employer')->get();
the query
// select * from "job_listings"
//select * from "employers" where "employers"."id" in (1,2,3,4,5) ...
-----

-----------
//configure app in
AppServiceProvider.php


public function boot() {
	Model::preventLazyLoading();
}

//prevent this code::: $jobs = Job::all();
//should be::: $jobs = Job::with('employer')->get();

--------------




=========DAY 14


$jobs = Job::with('employer')->paginate(2);

// pagination link
{{ jobs->links() }}


// no index pagination, previous and next only
$jobs = Job::with('employer')->simplePaginate(2);


// is also simplePaginate but with cursor="uniquestring"
$jobs = Job::with('employer')->cursorPaginate(2);


---------
//change pagination design
php artisan vendor:publish

laravel-pagination

public function boot() {
	Paginator::defaultView('viewname'); // custom
	Paginator::useBootstrapFive(); //bootstrap
}
--------


=========DAY 15

----------

php artisan migrate:fresh --seed

php artisan db:seed //default

php artisan db:seed --class=JobSeeder


Separate seeders.


DatabaseSeeder.php


public function run(): void {
	//...	
	
	$this->call(JobSeeder:class);
}
---------------


=========DAY 16
---------------------

wildcard route should be in bottom

Route::get('/jobs/create')
Route::get('/jobs/{id}')


dd(request()-all());
dd(request('key'));




Job::create([
	'title'=> request('title'),
	'salary'=> request('salary'),
	'employer_id'=> 1
]);

return redirect('/jobs');


//order by timestamp desc "latest" keyword
$jobs = Job::with('employer')->latest()->simplePaginate(2);


-------------------


=========== DAY 17

---------------

<button {{ $attributes->merge(['class' => '' ]) }}> {{ $slot }} </button>

<x-button href="/jobs/create">Create</x-button>  


@if($errors->any()) 

	@foreach($errors->all() as $error)
		<li>{{ error }}</li>
	@endforeach

@endif


--------------

========= DAY 18

-------------
Eloquent can be

$job['title']
or $job->title

:: FOR EDIT
// validate
// authorize
// update
// persist
// redirect


$job->title = request('title');

or 

$job->update([
	'title' => request('title'),
])


:: FOR DELETE
// authorize
// delete
// redirect


$job = Job::findOrFail($id);
$job->delete();

or inline

Job::findOrFail($id)->delete();


button with external form

<button form="form-name">Delete</button>

<form method="POST" action="/jobs/{{ $job->id}}" id="form-name" class"hidden">
	@csrf
	@method('DELETE')
</form>


-------------

========= DAY 19

-------------

Route::get('/jobs/{job}', function (Job $job) {
	return view('jobs.show', ['job' => $job]);
});

default is job:id

Route::get('/jobs/{job:id}', function (Job $job) {
	return view('jobs.show', ['job' => $job]);
});

Route::delete('/jobs/{job}', function (Job $job) {
	$job->delete();
});


short

Route::view('/', 'home');


php artisan route:list
php artisan route:list --except-vendor




Route:resource('jobs', JobController:class);
paired with
php artisan make:controller ControllerName --resource


~~~EXCEPT
Route:resource('jobs', JobController:class, [
	'except' =>['edit'];
]);

~~ONLY
Route:resource('jobs', JobController:class, [
	'only' =>['index', 'show', 'create', 'store'];
]);


------------


======= DAY 20

------------

Starter kits like breeze
login etc..

-Laravel Breeze
-Blade with Alpine
-Pest


one parameter: string, 'name'
more than one: array of string  ['name1', 'name2']


------------

======= DAY 21

---------
- Always put required attribute to required form inputs


@auth
	//code
@endauth


@guest
	//code
@endguest


-------

====== DAY 22

--------

// sample password rule validation

request()->validate([
	'password' => ['required', Password::min(5)->letters()->numbers()->max(10)],

]);



request()->validate([
	'password' => ['required', Password::min(6), 'confirmed'],

]);


User::create([
	'first_name' => request('first_name'),
]);


$attributes = request()->validate([
	....
]);

User::create($attributes);

$user = User::create($attributes);


Auth::login($user);

// redirect..



Auth::logout();

Auth::attempt($attributes);
//true or false


// recycle session token
$request()->session()->regenerate();



//Illuminate Validation : ValidationException
if (! Auth::attempt($attributes)) {
	throw ValidationException::withMessages([
		'email'=>'Message here.',
	]);
}


:value="old('email')"
or value="{{old('email')}}"



--------

=======DAY 23

AUTHORIZATION (6 steps)
---------


STEP 1: INLINE AUTHORIZATION

public function edit(Job $job) {
	if (Auth::guest()) {
		redirect('/');
	}

	return view('jobs.edit', ['job' => $job]);
}




Employer model

public function user(): BelongsTo {
	return $this->belongsTo(User::class);
}


$mode->is() = determine if two models have the same ID and belong to the same table.

reverse isNot

if ($job->employer->user->isNot(Auth::user())) {
	abort(403);	
}


STEP 2: GATES

Gate Facade

=== defined in controller
Gate::define('edit-job', function(User $user, Job $job) {
	return $job->employer->user->is($user);
});
===

//return 403
Gate::authorize('edit-job', $job);

alternative

if (Gate::denies('edit-job', $job)) {
	redirect('/');
}

STEP 3: DEFINE GATES INSIDE APPSERVICE PROVIDER

PUT GATES IN APP SERVICE PROVIDER

this one
==== defined in app service provider (global)
Gate::define('edit-job', function(User $user, Job $job) {
	return $job->employer->user->is($user);
});
===

NEW
public function edit(Job $job) {
	Gate::authorize('edit-job', $job);
	return view('jobs.edit', ['job' => $job]);
}


STEP 4: CAN

$model->can() = determine if the entity has the given abilities

reverse cannot

Another method, if user can edit job.
Auth::user()->can('edit-job', $job)

if (Auth::user()->cannot('edit-job', $job)) {
	dd('failure');
}



edit.blade.php

@can('edit-job', $job)
	//code here
@endcan


STEP 5: MIDDLEWARE

Route::resource('jobs', JobController::class)->only(['index','show])->middleware('auth);

or except


Route::resource('jobs', JobController::class)->except(['index','show])->middleware('auth);


FOR MORE READABILITY,
split the route resource to each line of codes
then add middleware for specific routes


Route::post('/jobs', [JobController::class, 'store'])->middleware('auth);

										can:gatename,relevant_column
Route::get('/jobs/{job}/edit', [JobController::class, 'edit'])->middleware(['auth', 'can:edit-job,job']);

or

Route::get('/jobs/{job}/edit', [JobController::class, 'edit'])->middleware('auth')->can('edit-job', 'job');


UPDATED
public function edit(Job $job) {
	return view('jobs.edit', ['job' => $job]);
}


STEP 6: POLICIES

php artisan make:policy JobPolicy -m Job

class JobPolicy
{
    public function edit(User $user, Job $job): bool
    {
        return $job->employer->user->is($user);
    }
}

// with policy

Route::get('/jobs/{job}/edit', [JobController::class, 'edit'])->middleware('auth')->can('edit', 'job');


All "can" now points to the policy

@can('edit', $job)
	//code here
@endcan




SIMPLE APP = STEP 2: GATE FACADE
LARGE APP = STEP 6: POLICIES

---------

========DAY 24

-----------

MAIL

php artisan make:mail JobPosted

sample code

Route::get('test', function() {
	//return new JobPosted();

	or
	Mail::to('abc@gmail.com')->send(
		new JobPosted()
	);

	return Done;
});


For production

APP_ENV=local
to APP_ENV=local

change email address and name source from .env file


Override from,  JobPosted mail

public function envelope(): Envelope {
	return new Envelope(
		subject: 'Job Posted',
		from: 'admin@gmail.com',
	);
}



mailtrap.io
5:54:13
https://www.youtube.com/watch?v=SqTdHCTWqks


JobController, store function

$job = Job::create([
	'title' => request('title'),
	'salary' => request('salary'),
	'employer_id' => 1,
]);

Mail::to($job->employer->user->email)->send(
	new JobPosted();
)

or 

//auto detect email
Mail::to($job->employer->user)->send(
	new JobPosted();
)

unfinished..








{{ url('/jobs/'.$job->id) }}

-----------

=========DAY 25


----------
Queue


Mail::to($job->employer->user)->queue(
	new JobPosted();
)

php artisan queue:work

Route::get('test', function() {
	dispatch(function() {
		logger("stored in laravel.log");
			
	});

	dispatch(function() {
		logger("stored in laravel.log");
			
	})->delay(5);

	return "done";

});

php artisan make:job TranslateJob

handle() {
	logger("stored in laravel.log");	
}


Route::get('test', function() {
	TranslateJob::dispatch();
	return "done";
});



---------

========
















