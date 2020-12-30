# Modules

Modules provide an alternate code organization architecture that scales well
with your ever increasing codebase. They are completely independent components
that reside in their own folders.

<h3>
    <p class="tip">Lighpack provides out of the box support for modules 👻</p>
</h3>

For example, when developing an ecommerce app along with a blog for better SEO promotion,
in the typical <code>MVC</code> fashion, you will end up having something similar code 
structure:

```
lightpack
    |--- app
        |--- controllers
            |--- TagController.php
            |--- BlogController.php
            |--- BrandController.php
            |--- ProductController.php
            |--- CategoryController.php
        |--- models
            |--- TagModel.php
            |--- BlogModel.php
            |--- BrandModel.php
            |--- ProductModel.php
            |--- CategoryModel.php
        |--- views
            |--- tag.php
            |--- blog.php
            |--- brand.php
            |--- product.php
            |--- category.php
```

There is no problem with above organization and infact the most recommended too. But
some of you might soon realize that organizing codebase based on features probably 
scales well and is easier to manage and distribute in the long run.

The same above features will end up in a codebase structure like this in a <code>modules</code> 
based approach.

```
lightpack
    |--- modules
        |--- Shop
            |--- controllers
                |--- BrandController.php
                |--- ProductController.php
                |--- CategoryController.php
            |--- models
                |--- BrandModel.php
                |--- ProductModel.php
                |--- CategoryModel.php
            |--- views
                |--- brand.php
                |--- product.php
                |--- category.php
        |--- Blog
            |--- controllers
                |--- TagController.php
                |--- BlogController.php
            |--- models
                |-- TagModel.php
                |-- BlogModel.php
            |--- views
                |--- tag.php
                |--- blog.php
```

As you can see, modules will start to make sense for really large codebases. So its
upto you to decide how you would like to organize your codebase.

<p class="tip">
You may start in the typical <code>MVC</code> fashion and once your project becomes
too big demanding multiple of features, you may decide to move with <code>modules</code>.
</p>