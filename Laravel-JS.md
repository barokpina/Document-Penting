## Laravel 

1. instalasi laravel terbaru 
- download composer 
- download node.js

```
composer global require laravel/installer

laravel new example-app
```
2. Koneksi di Mysql di Env 

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_crud_js
DB_USERNAME=root
DB_PASSWORD=
```

3. Buat Table Database 

```
php artisan make:migration create_products_table
```

4. Buka Produk Tabel 
```
public function up()
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->integer('price');
        $table->timestamps();
    });
}
```

```
php artisan migrate
```

5. Buat Model 
```
php artisan make:model Product
```

6. Buka model nya 
```
class Product extends Model
{
    protected $fillable = ['name', 'price'];
}
```
7. Buat controller ApI
```
php artisan make:controller ProductController
```
8. Buka Control ApI nya 
```
use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    // CREATE
    public function store(Request $request)
    {
        $product = Product::create([
            'name' => $request->name,
            'price' => $request->price
        ]);

        return response()->json($product);
    }

    // READ
    public function index()
    {
        return response()->json(Product::all());
    }

    // UPDATE
    public function update(Request $request, $id)
    {
        $product = Product::find($id);
        $product->update($request->all());

        return response()->json($product);
    }

    // DELETE
    public function destroy($id)
    {
        Product::destroy($id);
        return response()->json(['message' => 'Deleted']);
    }
}
```
9. Buat Route API 
```
use App\Http\Controllers\ProductController;

Route::get('/products', [ProductController::class, 'index']);
Route::post('/products', [ProductController::class, 'store']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
```
10. Bagian Front Enda 

```
<!DOCTYPE html>
<html>
<head>
    <title>CRUD JS + Laravel</title>
</head>
<body>

<h2>Tambah Produk</h2>

<input type="text" id="name" placeholder="Nama Produk">
<input type="number" id="price" placeholder="Harga">
<button onclick="save()">Simpan</button>

<h2>Data Produk</h2>
<ul id="list"></ul>

<script>
const API = "http://127.0.0.1:8000/api/products";

function save() {
    fetch(API, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            name: document.getElementById('name').value,
            price: document.getElementById('price').value
        })
    })
    .then(res => res.json())
    .then(() => load());
}

function load() {
    fetch(API)
    .then(res => res.json())
    .then(data => {
        let html = "";
        data.forEach(p => {
            html += `<li>
                ${p.name} - ${p.price}
                <button onclick="del(${p.id})">Hapus</button>
            </li>`;
        });
        document.getElementById('list').innerHTML = html;
    });
}

function del(id) {
    fetch(API + '/' + id, { method: 'DELETE' })
    .then(() => load());
}

load();
</script>

</body>
</html>

```
