# Laravel-Generate-PDF
PDF generate application using "Dompdf"  library


### ✔️Install the laravel-dompdf package 
```
composer require barryvdh/laravel-dompdf
```

### ✔️Configure the package in application
_config >> app.php_
```
'providers' => [
    ....
    Barryvdh\DomPDF\ServiceProvider::class,
],
'aliases' => [
    ....
    'PDF' => Barryvdh\DomPDF\Facade::class,
],
```

### ✔️Create a layout blade file

_Inside the resources >> views folder, create a new file called layout.blade.php_

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Laravel 6 CRUD Example</title>
  <link href="{{ asset('css/app.css') }}" rel="stylesheet" type="text/css" />
</head>
<body>
  <div class="container">
    @yield('content')
  </div>
  <script src="{{ asset('js/app.js') }}" type="text/js"></script>
</body>
</html>
```

### ✔️Create model and migration files
```
php artisan make:model Disneyplus
```

_[timestamp].create_disneypluses_table.php_
```
public function up()
{
        Schema::create('disneypluses', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('show_name');
            $table->string('series');
            $table->string('lead_actor');
            $table->timestamps();
        });
}
```

### ✔️Migrate the database
```
php artisan migrate
```

### ✔️Create a controller and routes 
```
php artisan make:controller DisneyplusController
```

_add the two routes inside the routes >> web.php_
```
Route::get('disneyplus', 'DisneyplusController@create')->name('disneyplus.create');
Route::post('disneyplus', 'DisneyplusController@store')->name('disneyplus.store');
```

_DisneyplusController.php_
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Disneyplus;
use PDF;

class DisneyplusController extends Controller
{
    public function create()
    {

    }

    public function store()
    {
        
    }
}
```

### ✔️Create a form blade file for input the data

_Create new form.blade.php in views/ directory_

```
@extends('layout')

@section('content')
<style>
  .uper {
    margin-top: 40px;
  }
</style>
<div class="card uper">
  <div class="card-header">
    Add Disneyplus Shows
  </div>
  <div class="card-body">
    @if ($errors->any())
      <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
              <li>{{ $error }}</li>
            @endforeach
        </ul>
      </div><br />
    @endif
      <form method="post" action="{{ route('disneyplus.store') }}">
          <div class="form-group">
              @csrf
              <label for="name">Show Name:</label>
              <input type="text" class="form-control" name="show_name"/>
          </div>
          <div class="form-group">
              <label for="price">Series :</label>
              <input type="text" class="form-control" name="series"/>
          </div>
          <div class="form-group">
              <label for="quantity">Show Lead Actor :</label>
              <input type="text" class="form-control" name="lead_actor"/>
          </div>
          <button type="submit" class="btn btn-primary">Create Show</button>
      </form>
  </div>
</div>
@endsection
```

### ✔️Store data in the database
_write the two functions inside the DisneyplusController.php file_

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Disneyplus;

class DisneyplusController extends Controller
{
    public function create()
    {
        return view('form');
    }

    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'show_name' => 'required|max:255',
            'series' => 'required|max:255',
            'lead_actor' => 'required|max:255',
        ]);
        Disneyplus::create($validatedData);
   
        return redirect('/disneyplus')->with('success', 'Disney Plus Show is successfully saved');
    }
}
```

_add the fillable fields inside the Disneyplus.php model file_
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Disneyplus extends Model
{
    protected $fillable = ['show_name', 'series', 'lead_actor'];
}
```

__After the above steps, you can check your application via__
```
http://localhost:8000/disneyplus
```

### ✔️Create a view file for display the data
```
// web.php

Route::get('disneyplus/list', 'DisneyplusController@index')->name('disneyplus.index');
```

_create a view file called list.blade.php file_
```
@extends('layout')
@section('content')
<table class="table table-striped">
  <thead>
    <th>ID</th>
    <th>Show Name</th>
    <th>Series</th>
    <th>Lead Actor</th>
    <th>Action</th>
  </thead>
  <tbody>
    @foreach($shows as $show)
    <tr>
      <td>{{$show->id}}</td>
      <td>{{$show->show_name}}</td>
      <td>{{$show->series}}</td>
      <td>{{$show->lead_actor}}</td>
    </tr>
    @endforeach
  </tbody>
</table>
@endsection
```

_add the code inside the index() function of DisneyplusController.php_
```
public function index()
{
        $shows = Disneyplus::all();

        return view('list', compact('shows'));
}
```

__After that, check the list via__
```
http://localhost:8000/disneyplus/list
```
### ✔️Create a route to download the pdf file
```
// web.php

Route::get('/downloadPDF/{id}','DisneyplusController@downloadPDF');
```

_update the list.blade.php file and add the Download PDF link_
```
@extends('layout')
@section('content')
<table class="table table-striped">
  <thead>
    <th>ID</th>
    <th>Show Name</th>
    <th>Series</th>
    <th>Lead Actor</th>
    <th>Action</th>
  </thead>
  <tbody>
    @foreach($shows as $show)
    <tr>
      <td>{{$show->id}}</td>
      <td>{{$show->show_name}}</td>
      <td>{{$show->series}}</td>
      <td>{{$show->lead_actor}}</td>
      <td><a href="{{action('DisneyplusController@downloadPDF', $show->id)}}">Download PDF</a></td>
    </tr>
    @endforeach
  </tbody>
</table>
@endsection
```

### ✔️Create pdf.blade.php file to design our pdf
_inside the views folder, create one file called pdf.blade.php file_
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <table class="table table-bordered">
    <thead>
      <tr>
        <td><b>Show Name</b></td>
        <td><b>Series</b></td>
        <td><b>Lead Actor</b></td>     
      </tr>
      </thead>
      <tbody>
      <tr>
        <td>
          {{$show->show_name}}
        </td>
        <td>
          {{$show->series}}
        </td>
        <td>
          {{$show->lead_actor}}
        </td>
      </tr>
      </tbody>
    </table>
  </body>
</html>
```

###  ✔️Write a controller function to download the PDF
```
// DisneyplusController.php

public function downloadPDF($id) {
        $show = Disneyplus::find($id);
        $pdf = PDF::loadView('pdf', compact('show'));
        
        return $pdf->download('disney.pdf');
}
```

After the above step, you can download PDF via clicking __"Download PDF"__ button 
```
http://localhost:8000/disneyplus/list
```
