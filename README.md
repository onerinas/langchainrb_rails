💎🔗 Langchain.rb for Rails
---
The fastest way to sprinkle AI ✨ on top of your Rails app. Add OpenAI-powered question-and-answering in minutes.

Available for paid consulting engagements! [Email me](mailto:andrei@sourcelabs.io).

![Tests status](https://github.com/andreibondarev/langchainrb_rails/actions/workflows/ci.yml/badge.svg?branch=main)
[![Gem Version](https://badge.fury.io/rb/langchainrb_rails.svg)](https://badge.fury.io/rb/langchainrb_rails)
[![Docs](http://img.shields.io/badge/yard-docs-blue.svg)](http://rubydoc.info/gems/langchainrb_rails)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/andreibondarev/langchainrb_rails/blob/main/LICENSE.txt)
[![](https://dcbadge.vercel.app/api/server/WDARp7J2n8?compact=true&style=flat)](https://discord.gg/WDARp7J2n8)
[![X](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%40rushing_andrei)](https://twitter.com/rushing_andrei)

## Dependencies

* Ruby 3.0+
* Postgres 11+

## Table of Contents

- [Installation](#installation)
- [Generators](#rails-generators)

## Installation

Install the gem and add to the application's Gemfile by executing:
```bash
bundle add langchainrb_rails
```

If bundler is not being used to manage dependencies, install the gem by executing:
```bash
gem install langchainrb_rails
```

## Configuration w/ [Pgvector](https://github.com/pgvector/pgvector) (requires Postgres 11+)

1. Run the Rails generator to add vectorsearch to your ActiveRecord model
```bash
rails generate langchainrb_rails:pgvector --model=Product --llm=openai
```

This adds required dependencies to your Gemfile, creates the `config/initializers/langchainrb_rails.rb` initializer file, database migrations, and adds the necessary code to the ActiveRecord model to enable vectorsearch.

2. Bundle and migrate
```bash
bundle install && rails db:migrate
```

3. Set the env var `OPENAI_API_KEY` to your OpenAI API key: https://platform.openai.com/account/api-keys
```ruby
ENV["OPENAI_API_KEY"]= 
```

5. Generate embeddings for your model
```ruby
Product.embed!
```

This can take a while depending on the number of database records.

## Usage

### Question and Answering
```ruby
Product.ask("list the brands of shoes that are in stock")
```

Returns a `String` with a natural language answer. The answer is assembled using the following steps:

1. An embedding is generated for the passed in `question` using the selected LLM.
2. We calculate a [cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity) to find records that most closely match your question's embedding.
3. A prompt is created using the question and the above records (their `#as_vector` representation )are added as context.
4. This prompt is passed to the LLM to generate an answer

### Similarity Search
```ruby
Product.similarity_search("t-shirt")
```

Returns ActiveRecord relation that most closely matches the `query` using vector search.

## Customization

### Changing the vector representation of a record

By default, embeddings are generated by calling the following method on your model instance:
```ruby
to_json(except: :embedding)
```

You can override this by defining an `#as_vector` method in your model:
```ruby
def as_vector
  { name: name, description: description, category: category.name, ... }.to_json
end
```

Re-generate embeddings after modifying this method:

```ruby
Product.embed!
```

## Rails Generators

### Pgvector Generator

```bash
rails generate langchainrb_rails:pgvector --model=Product --llm=openai
```

### Pinecone Generator - adds vectorsearch to your ActiveRecord model
```bash
rails generate langchainrb_rails:pinecone --model=Product --llm=openai
```

Available `--llm` options: `cohere`, `google_palm`, `hugging_face`, `llama_cpp`, `ollama`, `openai`, and `replicate`. The selected LLM will be used to generate embeddings and completions.

The `--model` option is used to specify which ActiveRecord model vectorsearch capabilities will be added to.

Pinecone Generator does the following:
1. Creates the `config/initializers/langchainrb_rails.rb` initializer file
2. Adds necessary code to the ActiveRecord model to enable vectorsearch
3. Adds `pinecone` gem to the Gemfile

