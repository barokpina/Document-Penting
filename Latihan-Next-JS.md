ini install ektension vscode = javascript anda typescript - node.js modules intelesense - es7+


## install dan informasi 
1. Install nex js : npm install next@latest react@latest react-dom@latest (Type screet -yess, tailwind css - yess , src directory -no, import alisan - no)
2. jalankan = npm run dev
3. folder utama app = untuk membuat halaman apapun -> dengan membuat folder nama halaman nya dan baut page.tsx (cth : folder = Home , page.tsx )

contoh : buat folder -> Posts -> buat file page.tsx isintya = 
```

import Link from "next/Link" #penghanti Href adalah Link di nex js 

const Posts = () = {
return (
)
<div>
  <h1> POSTINGAN PAGE </h1>
<Link href="/posts">POSTING PAGE</Link>
</div>
}
export default Posts
```
```
'use client'  #wajib menggunakan ini karena ini client side dimasukan di folder component 
```
4. pastikan membuka inspec element : klik kanan di browser inspec element dan buka tab network dan lihat resorce nya
5. bagian syles css = bisa menggunkana global.css -> untuk styling css secara global lebih untuk kayak navigasi, side bar.
6. menambahan module css di folder halamanya = contoh : postPage.modules.css yang isinya sama kayak css -> dan tambahkan  (import styles from "./postPage.module.css" - lebih enak lgsg tailwind
7. 
   





###Laravel 

1. instalasi laravel terbaru 
- download composer 
- download node.js

```
composer global require laravel/installer

laravel new example-app
```

