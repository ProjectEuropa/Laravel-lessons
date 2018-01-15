## Laravel の基本 todo リスト作成


## Laravelプロジェクト作成
php5.6の場合
```bash
composer create-project "laravel/laravel=5.4.*" tasklist
```

php7.0以降の場合
```bash
composer create-project "laravel/laravel=5.5.*" tasklist
```

## データベースの設定・作成
```bash
php artisan make:migration create_tasks_table --create=tasks
```

database/migrationsを以下の通りに編集する。

```php
//database/migrationsディレクトリ
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateTasksTable extends Migration
{
    /**
     * マイグレーション実行
     *
     * @return void
     */
    public function up()
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->timestamps();
        });
    }

    /**
     * マイグレーションの巻き戻し
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('tasks');
    }
}
```

データベース「task_list」を作成し、.envを編集する
```.env
DB_HOST=127.0.0.1
DB_DATABASE=task_list
DB_USERNAME=root
DB_PASSWORD=root

#xamppやmamppであれば、DB_USERNAMEとDB_PASSWORDの初期設定は「root」になっている。
```

データベース作成と.envの編集が完了したら以下のコマンド実行
```bash
php artisan migrate
```
Exceptionが発生した場合は、.envのDB_USERNAMEやDB_PASSWORD、DB_DATABASEの設定を見直す。


## モデル作成
以下のコマンド実行でTaskモデルが作成される。
```bash
php artisan make:model Task
```

作成されたファイルとコード
```php
//Appディレクトリ
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    //
}
```


## ルーティング
routes/web.phpを以下のように編集

```php
<?php

use App\Task;
use Illuminate\Http\Request;

/**
 * 全タスク表示
 */
Route::get('/', function () {
    $tasks = Task::orderBy('created_at', 'asc')->get();

    return view('tasks')->with('tasks', $tasks);
});

/**
 * 新タスク追加
 */
Route::post('/task', function (Request $request) {
    $validator = Validator::make($request->all(), [
        'name' => 'required|max:255',
    ]);

    if ($validator->fails()) {
        return redirect('/')
            ->withInput()
            ->withErrors($validator);
    }

    $task = new Task;
    $task->name = $request->name;
    $task->save();

    return redirect('/');
});

/**
 * 既存タスク削除
 */
Route::delete('/task/{id}', function ($id) {
    Task::findOrFail($id)->delete();

    return redirect('/');
});

```

## ビューの作成
resources/views/layouts/app.blade.phpを作成し、以下の通りに編集する。
```html
<!DOCTYPE html>
<html lang="ja">
    <head>
        <title>Laravel Quickstart - Basic</title>
        <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
    </head>

    <body>
        <div class="container">
            <nav class="navbar navbar-default">
                <!-- ナビバーの内容 -->
            </nav>
        </div>

        @yield('content')

        
    <script src="https://code.jquery.com/jquery-1.12.4.min.js" integrity="sha256-ZosEbRLbNQzLpnKIkEdrPv7lOy9C27hHQ+Xp8a4MxAQ="
        crossorigin="anonymous"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
    </body>
</html>
```

resources/views/tasks.blade.php作成し、以下の通りに編集する。
```html
@extends('layouts.app')

@section('content')
    <div class="container">
        <div class="col-sm-offset-2 col-sm-8">
            <div class="panel panel-default">
                <div class="panel-heading">
                    New Task
                </div>

                <div class="panel-body">
                    <!-- Display Validation Errors -->
                    @include('common.errors')

                    <!-- New Task Form -->
                    <form action="{{ url('task')}}" method="POST" class="form-horizontal">
                        {{ csrf_field() }}

                        <!-- Task Name -->
                        <div class="form-group">
                            <label for="task-name" class="col-sm-3 control-label">Task</label>

                            <div class="col-sm-6">
                                <input type="text" name="name" id="task-name" class="form-control" value="{{ old('task') }}">
                            </div>
                        </div>

                        <!-- Add Task Button -->
                        <div class="form-group">
                            <div class="col-sm-offset-3 col-sm-6">
                                <button type="submit" class="btn btn-default">
                                    <i class="fa fa-btn fa-plus"></i>Add Task
                                </button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>

            <!-- Current Tasks -->
            @if (count($tasks) > 0)
                <div class="panel panel-default">
                    <div class="panel-heading">
                        Current Tasks
                    </div>

                    <div class="panel-body">
                        <table class="table table-striped task-table">
                            <thead>
                                <th>Task</th>
                                <th>&nbsp;</th>
                            </thead>
                            <tbody>
                                @foreach ($tasks as $task)
                                    <tr>
                                        <td class="table-text"><div>{{ $task->name }}</div></td>

                                        <!-- Task Delete Button -->
                                        <td>
                                            <form action="{{ url('task/'.$task->id) }}" method="POST">
                                                {{ csrf_field() }}
                                                {{ method_field('DELETE') }}

                                                <button type="submit" class="btn btn-danger">
                                                    <i class="fa fa-btn fa-trash"></i>Delete
                                                </button>
                                            </form>
                                        </td>
                                    </tr>
                                @endforeach
                            </tbody>
                        </table>
                    </div>
                </div>
            @endif
        </div>
    </div>
@endsection
```

## 参考
https://readouble.com/laravel/5.1/ja/quickstart.html

