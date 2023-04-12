# ChatGPT-API-embeddings-Supabase

## Why learn embeddings?

Learned from [RabbitHoleSyndrome](https://www.youtube.com/watch?v=Yhtjd7yGGGA&t=466s&ab_channel=RabbitHoleSyndrome) YouTube channel for introducing me to this new tech right from the beginning. I finally have the opportunity to work on and gain a deeper understanding of how this "magic" tech works behind the scenes this time.

It is basic features for scrap text build, learn from [mckaywrigley](https://www.youtube.com/watch?v=RM-v7zoYQo0&t=1060s) YouTube channel. Also there is advanced version (more UI refined) in his GitHub repo [here](https://github.com/mckaywrigley/paul-graham-gpt).

This is just the beginning journey in using base embeddings. With the techniques you've learned so far, you now have a powerful set of tools that can be applied to create diverse datasets. You can scrape text from various sources such as PDFs, websites, or even transcribe audio from podcasts or YouTube videos. **the possibilities are endless✨** Use our imagination.🙄🧐

## Technologies include

OpenAI embeddings; Supabase; Next.js; Data search and similarity functions build in Supabase; Streaming data and building a basic user interface with answer props animation.

## Learning Process

This tutorial demonstrates the step-by-step process of scraping and embedding text (include fetching, cleaning, and storing data), creating datasets, streaming data, and building a basic user interface (search and chat interfaces).

More details about work flow, you can check tutorial GitHub repo [here](https://github.com/mckaywrigley/paul-graham-gpt)

## What your need?

Create a .env.local file in the root of the repo with the following variables:

```
OPENAI_API_KEY=

NEXT_PUBLIC_SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=
```

For those curious about costs, the GPT embeddings charge was economical for this project. With approximately 3387 records across 34 pages in a Supabase table, the total cost was less than 20 cents (Canadian dollars) and Supabase is free for 2 projects! 💰

## So what you can get from here? 🤔

( I am always a database lover 🧡 )

1. Set up Supabase
   Run that in the SQL editor in Supabase as directed.<br>
   **pgvector** is a PostgreSQL _extension_ for _vector_ similarity search.

```plpgsql
   create extension vector;
```

2. Create the table

```plpgsql
create table paul_graham (
 id bigserial primary key,
 essay_title text,
 essay_url text,
 essay_date text,
 content text,
 content_tokens bigint,
 embedding vector (1536)
)
```

Enable RLS

3. Create data search function

```plpgsql
create or replace function paul_graham_search (
  query_embedding vector(1536),
  similarity_threshold float,
  match_count int
)
returns table (
  id bigint,
  essay_title text,
  essay_url text,
  essay_date text,
  content text,
  content_tokens bigint,
  similarity float
)
language plpgsql
as $$
begin
  return query
  select
    paul_graham.id,
    paul_graham.essay_title,
    paul_graham.essay_url,
    paul_graham.essay_date,
    paul_graham.content,
    paul_graham.content_tokens,
    1-(paul_graham.embedding <=> query_embedding) as similarity
from paul_graham
where 1 - (paul_graham.embedding <=> query_embedding) > similarity_threshold
order by paul_graham.embedding <=> query_embedding
limit match_count;
end;
$$;
```

NOTE: insert json from local to supabase, the numerical representations (1536) of text that capture the context of words in a document

4. Create index with PG-vector on the database, increase the performance of our similarity search what we doing.

```plpgsql
create index on paul_graham
using ivfflat (embedding vector_cosine_ops)
with (lists = 100)
```

HAVE A FUN PLAYING WITH SUPABASE! 💚
