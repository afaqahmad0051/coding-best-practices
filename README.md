![Laravel best practices](/images/logo-english.png?raw=true)

## Contents

[Single responsibility principle](#single-responsibility-principle)

[Methods should do just one thing](#methods-should-do-just-one-thing)

[Validation](#validation)

[Business logic should be in service class](#business-logic-should-be-in-service-class)

### **Single responsibility principle**

A class should have only one responsibility.

Bad:

```php
public function update(Request $request): string
{
    $validated = $request->validate([
        'title' => 'required|max:255',
        'posts' => 'required|array:date,type'
    ]);

    foreach ($request->posts as $post) {
        $date = $this->carbon->parse($post['date'])->toString();

        $this->logger->log('Update post ' . $date . ' :: ' . $);
    }

    $this->post->PostEvent($request->validated());

    return back();
}
```

Good:

```php
public function update(UpdateRequest $request): string
{
    $this->logService->logPosts($request->posts);

    $this->post->updateGeneralPost($request->validated());

    return back();
}
```

[🔝 Back to contents](#contents)

### **Methods should do just one thing**

A function should do just one thing and do it well.

Bad:

```php
public function getFullNameAttribute(): string
{
    if (Auth::user() && Auth::user()->hasRole('client') && Auth::user()->isVerified()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}
```

Good:

```php
public function getFullNameAttribute(): string
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient(): bool
{
    return Auth::user() && Auth::user()->hasRole('client') && Auth::user()->isVerified();
}

public function getFullNameLong(): string
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort(): string
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```

[🔝 Back to contents](#contents)

### **Validation**

Move validation from controllers to Request classes.

Bad:

```php
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

    ...
}
```

Good:

```php
public function store(PostRequest $request)
{
    ...
}

class PostRequest extends Request
{
    public function rules(): array
    {
        return [
            'title' => ['required', 'unique:posts', 'max:255'],
            'body' => ['required'],
            'publish_at' => ['nullable', 'date'],
        ];
    }
}
```

[🔝 Back to contents](#contents)

### **Business logic should be in service class**

A controller must have only one responsibility, so move business logic from controllers to service classes.

Bad:

```php
public function store(Request $request)
{
    if ($request->hasFile('image')) {
        $request->file('image')->move(public_path('images') . 'temp');
    }
    
    ...
}
```

Good:

```php
public function store(Request $request)
{
    $this->articleService->handleUploadedImage($request->file('image'));

    ...
}

class ArticleService
{
    public function handleUploadedImage($image): void
    {
        if (!is_null($image)) {
            $image->move(public_path('images') . 'temp');
        }
    }
}
```

[🔝 Back to contents](#contents)