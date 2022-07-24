---
title: "What I Learned: IKEA has an API"
date: 2020-12-29T13:41:12+01:00
tags: ["whatilearned"]
draft: false
---

Recently I tried to get my hands on a standing desk since I spend a
considerable time in the home office sitting in front of a computer. This is a
challenging endeavor in the current situation (it's late 2020, in the middle of
the "second wave" of the COVID-19 pandemic). The model that I plan to buy is
out of stock on the German IKEA homepage and can't be ordered. There is no
information on when it will be available again on the site itself.

Luckily, I learned that IKEA offers a REST API that can be accessed via
```
http://www.ikea.com/de/de/iows/catalog/availability/12345678
```

For the US, replace `de/de` with `en/us` and the eight digits at the end with
an item number.  Performing a GET request on that URL yields some XML (sigh),
that looks like this:

```xml
[...]
 <localStore buCode="124" timeZoneOffsetInMillis="3600000">
      <stock>
        <partNumber>12345678</partNumber>
        <isMultiProduct>false</isMultiProduct>
        <isSoldInStore>true</isSoldInStore>
        <isInStoreRange>true</isInStoreRange>
        <restockDate>2021-01-22</restockDate>
        <isValidForNotification>true</isValidForNotification>
        <availableStock>0</availableStock>
        <inStockProbabilityCode>LOW</inStockProbabilityCode>
        <validDate>2020-12-29</validDate>
        <findItList>
          <findIt>
            <partNumber>12345678</partNumber>
            <quantity>1</quantity>
            <type>CONTACT_STAFF</type>
          </findIt>
        </findItList>
      </stock>
    </localStore>
[...]
```
Note that I use the tool `tidy` to format the one-liner XML to something
readable. Here, you see all the good information the IKEA backend has to offer.
In particular, these entries are of high interest:

* availableStock: How many items are in stock at this store
* inStockProbabilityCode: Probability to get this item in the store right now
* restockDate: When a planned restock will take place

Mixing an API call with `tidy` and `grep` will work just fine:
```
curl -L http://www.ikea.com/de/de/iows/catalog/availability/12345678 | tidy -xml -iq - | grep availableStock
```
The output is one line per store with the items left in stock:
```xml
[...]
        <availableStock>0</availableStock>
        <availableStock>0</availableStock>
        <availableStock>0</availableStock>
        <availableStock>0</availableStock>
        <availableStock>0</availableStock>
        <availableStock>0</availableStock>
[...]
```
You can also get an overview of when different stores will be restocked.

Acknowledgments go to [another blog post](https://medium.com/@JoshuaAJung/api-of-the-day-ikea-availability-checks-8678794a9b52), that taught me about the IKEA API.
