---
layout:     post
title:      Careful with that monkey
date:       2017-13-09
summary:    Metaprogramming and death
categories: [ruby, rails, dynamic-languages]
---

Setting `Timeliness.default_timezone = :current` broke our app. On being saved to the DB, some dates were being decremented by a day. Not all dates though! Just the dates being saved to the Active Record 'store', i.e. saved a ruby hash serialised to a string in a VARCHAR column; something that looks like:

```rb
[6] pry(main)> ActiveRecord::Base.connection.select_all("select type_fields from registrations where id = 1209")
   (1.1ms)  EXEC sp_executesql N'select type_fields from registrations where id = 1209'
=> #<ActiveRecord::Result:0x007fd8e80c9838
 @column_types={},
 @columns=["type_fields"],
 @hash_rows=nil,
 @rows=
  [["--- !ruby/hash:ActiveSupport::HashWithIndifferentAccess\nnomination_type: New Facility\nassigned_to_id: '31'\nfacility_new_name: Debugging time!\nfocal_point_inspector_id: '71'\nstate_id: '5'\nexpected_date_of_operation: ''\nnominator_type: Title Holder\nsignature_date: 2017-09-11 16:00:00.000000000 Z\nreview_date: !ruby/object:DateTime 2017-09-13 00:00:00.000000000 +10:00\n"]]>
"--- !ruby/hash:ActiveSupport::HashWithIndifferentAccess\nnomination_type: New Facility\nassigned_to_id: '31'\nfacility_new_name: Debugging time!\nfocal_point_inspector_id: '71'\nstate_id: '5'\nexpected_date_of_operation: ''\nnominator_type: Title Holder\nsignature_date: 2017-09-12 00:00:00.000000000 +10:00\nreview_date: !ruby/object:DateTime 2017-09-13 00:00:00.000000000 +10:00\n"
```

Weird, because this config changed shouldn't have affected the app: the previous value of Timeliness.default_timezone just instructed Timeliness to use the system timezone, which was exactly the same as the `:current` timezone.

Some digging revealed that previously, the value was being stored to the DB as

```
2017-09-12 00:00:00.000000000 +10:00
```

After the config change, the same date was being recorded as

``` ruby
2017-09-11 16:00:00.000000000 Z
```

*Further* digging uncovered a monkey patch to `ActiveRecord::Base`:

```rb
module DatesThatWork
  extend ActiveSupport::Concern

  module ClassMethods
    def define_attr_accessors_for(field, type, format)
      define_method("#{field}_string=") do |value|
        @value_cache ||= {}
        @value_cache[field] = value
        parsed_value = Timeliness.parse(value, type)
        debugger
        parsed_value = parsed_value.beginning_of_day if parsed_value && type == :date
        send("#{field}=", parsed_value)
      end

      define_method("#{field}_string") do
        @value_cache ||= {}
        debugger
        if send(field).nil?
          @value_cache[field]
        else
          send(field).strftime(format)
        end
      end
    end
  end
end
```

The important line is

``` ruby
        parsed_value = Timeliness.parse(value, type)
```

With the original config setting (the default, `Timeliness.default_timezone = XXX`), `parsed_value` had type `Time`. With the new config setting, `parsed_value` had type `ActiveSupport::TimeWithZone`.
