![Laravel best practices](/images/logo-english.png?raw=true)

## Contents

[Single responsibility principle](#single-responsibility-principle)

[Validation](#validation)

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