---
layout: post
title: Async Queries in a Web Service
description: The affects of async database calls on a Web API.
categories: Concurrency .Net Databases
tags: Async .Net Databases
---

In this post I’m going to be writing about using async queries using the data from [Ways to load a file to postgres]({% link _posts/2018-08-14-Ways-to-load a-file-to-Postgres.md %}). I worked with async database calls there as well, but this time I wanted to see what effect it would have on a web service. I took a similar approach to what I did in [Why async is important to a web service]({% link _posts/2018-07-23-why-async-is-important-to-a-web-service.md %}). There is an API and a client, the client calls either the sync or async API and reports timings and error counts. The API is simple, one endpoint to get all the keys, then two APIs to get details about each anime, sync and async. The code for this is simple.

```csharp
public Anime Get(int id)
{
    using (var connection = new NpgsqlConnection(configuration["connectionString"]))
    {
        connection.Open();
        using (var command = connection.CreateCommand())
        {
            var result = new Anime();
            command.CommandText = "select anime.id, anime.name, type.name, episodes, rating, members from anime join type on type_id = type.id where anime.id = @id";
            command.Parameters.Add(new NpgsqlParameter("id", id));
            using (var reader = command.ExecuteReader())
            {
                if (reader.Read())
                {
                    result.Id = reader.GetInt32(0);
                    result.Name = reader.GetString(1);
                    result.Type = reader.GetString(2);
                    result.Episodes = reader.IsDBNull(3) ? (int?)null : reader.GetInt32(3);
                    result.Rating = reader.IsDBNull(4) ? (double?)null : reader.GetDouble(4);
                    result.Members = reader.IsDBNull(5) ? (int?)null : reader.GetInt32(5);
                }
            }
            result.Genres = new List<string>();
            command.CommandText = "select genre.name from anime_genre join genre on anime_genre.genre_id = genre.id where anime_id = @id";
            using (var reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    result.Genres.Add(reader.GetString(0));
                }
            }
            return result;
        }
    }
}

public async Task<Anime> GetAsync(int id)
{
    using (var connection = new NpgsqlConnection(configuration["connectionString"]))
    {
        await connection.OpenAsync();
        using (var command = connection.CreateCommand())
        {
            var result = new Anime();
            command.CommandText = "select anime.id, anime.name, type.name, episodes, rating, members from anime join type on type_id = type.id where anime.id = @id";
            command.Parameters.Add(new NpgsqlParameter("id", id));
            using (var reader = await command.ExecuteReaderAsync())
            {
                if (await reader.ReadAsync())
                {
                    result.Id = reader.GetInt32(0);
                    result.Name = reader.GetString(1);
                    result.Type = reader.GetString(2);
                    result.Episodes = reader.IsDBNull(3) ? (int?)null : reader.GetInt32(3);
                    result.Rating = reader.IsDBNull(4) ? (double?)null : reader.GetDouble(4);
                    result.Members = reader.IsDBNull(5) ? (int?)null : reader.GetInt32(5);
                }
            }
            result.Genres = new List<string>();
            command.CommandText = "select genre.name from anime_genre join genre on anime_genre.genre_id = genre.id where anime_id = @id";
            using (var reader = await command.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    result.Genres.Add(reader.GetString(0));
                }
            }
            return result;
        }
    }
}
```

The results of running it look like this.

Count | sync (s) | sync errors | async (s) | async errors
--- | --- | --- | --- | ---
1,000 | .5 | 0 | .5 | 0
2,000 | 1 | 0 | 1.1 | 0
3,000 | 1.7 | 1 | 1.7 | 0
4,000 | 2.4 | 2 | 2.4 | 0
5,000 | 3.2 | 3 | 2.9 | 0
6,000 | 3.8 | 8 | 3.7 | 0

The total time for both is within a few tenths of a second for each, not enough to declare either one faster. However, the sync version started getting errors around 3,000 connections. The errors indicate that the sync version is running into a limit on the number of simultaneous
connections and time outs. With the async version it was able to handle a little over twice as many before throwing errors.

## Summary

Based on [Why async is important to a web service]({% link _posts/2018-07-23-why-async-is-important-to-a-web-service.md %}) this is the expected result. Making the calls async doesn’t change how fast (or slow) the data is retrieved from the database, but how efficiently the API can process the requests. An individual user won’t see any difference in response time, however the same server will be able to handle significantly more request if the database calls are async. Considering the code isn’t that much different between the two I think it is well worth the effort.

The code for this post is at <https://github.com/jamesdalton/async-queries-in-a-webservice>