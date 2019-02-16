# Flutter Paginator

A customizable pagination package for Flutter.

## Getting Started

This project is a starting point for a Dart
[package](https://flutter.io/developing-packages/),
a library module containing code that can be shared easily across
multiple Flutter or Dart projects.

For help getting started with Flutter, view our 
[online documentation](https://flutter.io/docs), which offers tutorials, 
samples, guidance on mobile development, and a full API reference.

## Screenshots

<img src="https://github.com/UdaraWanasinghe/FlutterPaginator/blob/master/screenrecord.gif" height="480em"/>

## Using

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:flutter_paginator_example/flutter_paginator/enums.dart';
import 'package:http/http.dart' as http;

import 'flutter_paginator/flutter_paginator.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Paginator',
      home: HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return HomeState();
  }
}

class HomeState extends State<HomePage> {
  GlobalKey<PaginatorState> paginatorGlobalKey = GlobalKey();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Flutter Paginator'),
        actions: <Widget>[
          IconButton(
            icon: Icon(Icons.format_list_bulleted),
            onPressed: () {
              paginatorGlobalKey.currentState
                  .changeState(listType: ListType.LIST_VIEW);
            },
          ),
          IconButton(
            icon: Icon(Icons.grid_on),
            onPressed: () {
              paginatorGlobalKey.currentState.changeState(
                listType: ListType.GRID_VIEW,
                gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                    crossAxisCount: 2),
              );
            },
          ),
          IconButton(
            icon: Icon(Icons.library_books),
            onPressed: () {
              paginatorGlobalKey.currentState
                  .changeState(listType: ListType.PAGE_VIEW);
            },
          ),
        ],
      ),
      body: Paginator.listView(
        key: paginatorGlobalKey,
        pageLoadFuture: sendCountriesDataRequest,
        pageItemsGetter: listItemsGetter,
        listItemBuilder: listItemBuilder,
        loadingWidgetBuilder: loadingWidgetMaker,
        errorWidgetBuilder: errorWidgetMaker,
        emptyListWidgetBuilder: emptyListWidgetMaker,
        totalItemsGetter: totalPagesGetter,
        pageErrorChecker: pageErrorChecker,
        scrollPhysics: BouncingScrollPhysics(),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          paginatorGlobalKey.currentState.changeState(
              pageLoadFuture: sendCountriesDataRequest, resetState: true);
        },
        child: Icon(Icons.refresh),
      ),
    );
  }

  Future<CountriesData> sendCountriesDataRequest(int page) async {
    try {
      String url = Uri.encodeFull(
          'http://api.worldbank.org/v2/country?page=$page&format=json');
      http.Response response = await http.get(url);
      return CountriesData.fromResponse(response);
    } catch (e) {
      if (e is IOException) {
        return CountriesData.withError(
            'Please check your internet connection.');
      } else {
        print(e.toString());
        return CountriesData.withError('Something went wrong.');
      }
    }
  }

  List<dynamic> listItemsGetter(CountriesData countriesData) {
    List<String> list = [];
    countriesData.countries.forEach((value) {
      list.add(value['name']);
    });
    return list;
  }

  Widget listItemBuilder(value, int index) {
    return ListTile(
      leading: Text(index.toString()),
      title: Text(value),
    );
  }

  Widget loadingWidgetMaker() {
    return Container(
      alignment: Alignment.center,
      height: 160.0,
      child: CircularProgressIndicator(),
    );
  }

  Widget errorWidgetMaker(CountriesData countriesData, retryListener) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Padding(
          padding: const EdgeInsets.all(16.0),
          child: Text(countriesData.errorMessage),
        ),
        FlatButton(
          onPressed: retryListener,
          child: Text('Retry'),
        )
      ],
    );
  }

  Widget emptyListWidgetMaker(CountriesData countriesData) {
    return Center(
      child: Text('No countries in the list'),
    );
  }

  int totalPagesGetter(CountriesData countriesData) {
    return countriesData.total;
  }

  bool pageErrorChecker(CountriesData countriesData) {
    return countriesData.statusCode != 200;
  }
}

class CountriesData {
  List<dynamic> countries;
  int statusCode;
  String errorMessage;
  int total;
  int nItems;

  CountriesData.fromResponse(http.Response response) {
    this.statusCode = response.statusCode;
    List jsonData = json.decode(response.body);
    countries = jsonData[1];
    total = jsonData[0]['total'];
    nItems = countries.length;
  }

  CountriesData.withError(String errorMessage) {
    this.errorMessage = errorMessage;
  }
}

```