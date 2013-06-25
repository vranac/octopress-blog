---
comments: true
date: 2011-08-25 16:03:58+00:00
layout: post
slug: symfony2-sonata-admin-bundle-and-file-uploads
title: Symfony2, Sonata Admin Bundle and file uploads
wordpress_id: 375
categories:
- PHP
- Symfony 2
tags:
- File Upload
- PHP
- Sonata
- Symfony 2
published: true
comments: false
---

I am writing a bundle that needs to have a file uploaded to server and later served, I am also using Sonata Admin bundle.
<!--more-->
First thing I needed to understand is that I will need an additional variable in my entity that will be used for the FileType object and that variable **WILL NOT BE MAPPED** to doctrine.

Class is shorten for brevity

```php
class Product
{
  protected $id;
...
  protected $imageName;

  protected $file;
...
  public function getAbsolutePath()
  {
      return null === $this->imageName ? null : $this->getUploadRootDir().'/'.$this->imageName;
  }

  public function getWebPath()
  {
    return null === $this->imageName ? null : $this->getUploadDir().'/'.$this->imageName;
  }

  protected function getUploadRootDir($basepath)
  {
    // the absolute directory path where uploaded documents should be saved
    return $basepath.$this->getUploadDir();
  }

  protected function getUploadDir()
  {
    // get rid of the __DIR__ so it doesn't screw when displaying uploaded doc/image in the view.
    return 'uploads/products';
  }

  public function upload($basepath)
  {
    // the file property can be empty if the field is not required
    if (null === $this->file) {
        return;
    }

    if (null === $basepath) {
        return;
    }

    // we use the original file name here but you should
    // sanitize it at least to avoid any security issues

    // move takes the target directory and then the target filename to move to
    $this->file->move($this->getUploadRootDir($basepath), $this->file->getClientOriginalName());

    // set the path property to the filename where you'ved saved the file
    $this->setImageName($this->file->getClientOriginalName());

    // clean up the file property as you won't need it anymore
    $this->file = null;
  }
}
```

As you can see it is pretty straight forward and you can see it in the [Manual](http://symfony.com/doc/2.0/cookbook/doctrine/file_uploads.html) as well.

I made a few changes to the way path is generated, I did not like the way it was done in the book, so I pass the base dir path to the class, easier for testing, and less hassle if it needs to be implemented elsewhere.

So far, so good, nothing especially new here.

It's time for the magic trick :)

Class shortened for brevity

```php
class Product extends Admin {
...
  protected function configureFormFields(FormMapper $formMapper) {
    $formMapper
            ->with('General')
...
            ->add('file', 'file', array('required' => false))
...
            ->end()
    ;
  }
...
  public function prePersist($product) {
    $this->saveFile($product);
  }

  public function preUpdate($product) {
    $this->saveFile($product);
  }

  public function saveFile($product) {
    $basepath = $this->getRequest()->getBasePath();
    $product->upload($basepath);
  }
}
```

What I did not understand at first is that I can use any field type defined in Symfony2 not only those types defined by the Sonata Admin Bundle.

Once I had an a-ha moment, I just added the file type (line 8, second parameter) and done, I had a nice file upload field displayed.

Sonata admin bundle exposes the pre/post persist and pre/post update calls for us to use.

Because we need the file moved and named **BEFORE** persisting the entity to the DB we need to use the pre* calls,
prePersist call for the Create and preUpdate call for the Update/Edit.

With that I had a final piece of the puzzle.

I added a function that would take the $product param (which is a Product Entity), so that we do not have code duplication.
The base dir path is retrieved from the request, and passed into the upload function of the Product Entity.

Once you hit save button on the form, the appropriate pre* call will be made, file will be uploaded and moved to your location under the web directory, and the entity will be persisted into the db.



