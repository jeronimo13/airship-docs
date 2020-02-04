---
id: ruby
title: Ruby API Reference
sidebar_label: Ruby
---

## Flagger
### init

```ruby
def self.init(init_args)
```

`init` method gets `FlaggerConfiguration`, establishes and maintains SSE connections and initialize Ingester

> Note: `init` must be called only once, at the start of your application. 
>Your program __must__ wait for `init` to finish before using any other `Flagger` methods

```ruby
api_key = 'x2ftC7QtG7arQW9l'
source_url = "https://flagger.notairshiphq.com"
sse_url = "https://sse.notairshiphq.com"
ingestion_url = "https://ingestion.notairshiphq.com"
backup_url = "https://backup-api.airshiphq.com"
log_level = "warn"
args = InitArguments::new(api_key, {log_level: log_level, source_url: source_url, sse_url: sse_url,
                                  ingestion_url: ingestion_url, backup_url: backup_url})
Flagger.init(args)
```

| name            | type   | Required | Default                           | Description                                                                                             |
| --------------- | ------ | -------- | --------------------------------- | ------------------------------------------------------------------------------------------------------- |
| api_key         | string | true     | None                              | API key to an environment                                                                               |
| source_url      | string | false    | https://api.airshiphq.com/        | URL to get `FlaggerConfiguration`                                                                         |
| backup_url      | string | false    | https://backup-api.airshiphq.com/ | backup URL to get `FlaggerConfiguration`                                                                  |
| sse_url         | string | false    | https://sse.airshiphq.com/        | URL for real-time updates of `FlaggerConfiguration` via sse                                                                       |
| ingestion_url   | string | false    | https://ingestion.airshiphq.com   | URL for ingestion                                                                                       |
| log_lvl         | string | false    | ERROR                             | set up log level: ERROR, WARN, DEBUG. Debug is the most verbose level and includes all Network requests |

- If `api_key` is not provided `init` throws FlaggerInitializationException
- If not provided default arguments values are used and printed to Debug
- If second(third …) call of `init` happens:
    - If the arguments are the same, `init` method does nothing
    - If arguments differ, `Flagger` prints warnings and recreates(closes and creates new) resources(SSE connection, 
    Ingester, gets new `FlaggerConfiguration`).
    - > Note: you must call init only once
- If initial `FlaggerConfiguration` is not fetched from source/backup than print Warning
- If `Flagger` fails to get `FlaggerConfiguration` then all Flags Functions return [Default Variation](../flagger-sdk/default-variation.md)
- If SSE connection fails than print Warning and retry until connection is established
- If you call any Flag Function BEFORE `init` is finished then you'll get [Default Variation](../flagger-sdk/default-variation.md)  


### shutdown

```ruby
def self.shutdown(timeout)
```

`shutdown` ingests data(if any), stop ingester and closes SSE connection.
`shutdown` waits to finish current ingestion request, but no longer than a timeoutMillis.

returns `true` if closed by timeout 

> Note: you __must__ call shutdown only once before the end of the application runtime. 

```ruby
Flagger.shutdown(1000)
```

### publish

```ruby
  def self.publish(entity)
```

Explicitly notify Airship about an Entity

```ruby
Flagger.publish(IdEntity::new("42"))
```

### track

```ruby
  def self.track(name, event_properties, entity)
```

Simple event tracking API.
Entity is an optional parameter if it was set before.

```ruby
Flagger.track('test', {:age => 40}, Entity::new("42", Hash::new))
```

### set_entity

```ruby
  def self.set_entity(entity)
```

`set_entity` stores an entity in Flagger, which allows omission of entity in other API methods. 

```ruby
Flagger.set_entity(Entity::new("42", Hash::new))

# here we are omitting, because Flagger has already stored "entity"
Flagger.flag_is_enabled('show_wallet')
```

>If you don't provide __any__ entity to Flagger:
>- flag functions always resolve with the default variation
>- `track` method doesn't record an event

Rule of thumb: make sure you provided an entity to the Flagger

## Flag Functions
### flag_is_enabled

```ruby
  def self.flag_is_enabled(codename, *entity)
```

Determines if flag is enabled for entity.

```ruby
is_enabled= Flagger.flag_is_enabled('show_wallet', IdEntity::new("42"))
```

Group example:

```ruby
company = Entity::new '42', :group => (Entity::new '4242', :type => "Company")
is_enabled= Flagger.flag_is_enabled('show_wallet', company)
```


### flag_is_sampled

```ruby
  def self.flag_is_sampled(codename, *entity)
```

Determines if entity is within the targeted subpopulations

```ruby
is_sampled= Flagger.flag_is_sampled('show_wallet', IdEntity::new("42"))
```

Group example:

```ruby
company = Entity::new '42', :group => (Entity::new '4242', :type => "Company")
is_sampled= Flagger.flag_is_sampled('show_wallet', company)
```

### flag_get_variation

```ruby
  def self.flag_get_variation(codename, *entity)
```

Returns the variation assigned to the entity in a multivariate flag

```ruby
variation = Flagger.flag_get_variation("show_wallet", IdEntity::new("42"))
```

Group example:

```ruby
company = Entity::new '42', :group => (Entity::new '4242', :type => "Company")
variation = Flagger.flag_get_variation("show_wallet", company)
```

### flag_get_payload

```ruby
  def self.flag_get_payload(codename, *entity)
```

Returns the payload associated with the treatment assigned to the entity

```ruby
payload = Flagger.flag_get_payload('show_wallet', IdEntity::new("42"))
```

Group example:

```ruby
company = Entity::new '42', :group => (Entity::new '4242', :type => "Company")
payload = flagger.flag_get_payload("show_wallet", company)
```