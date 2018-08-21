---
layout: post
title: '경유지를 고려한 대중교통 기반 최적 경로 도출 모델 개발'
author: "맵지"
date: "2018.06.08"
categories: Contest
cover: "/assets/contest/independent.bmp"
---

이 코드는 2018 대한지리학회 연례학술대회 포스터 분야에 제출한 '경유지를 고려한 대중교통 기반 최적 경로 도출 모델 개발'에서 사용한 코드이다. 2018학년도 1학기에 맵지(이영호, 이창규, 이유빈, 홍혁진)라는 팀명으로 독립연구에 참여하였으며, 프로젝트 주제 중의 하나였던 '경유지를 고려한 대중교통 기반 최적 경로 도출 모델 개발'을 진행하기 위해 다 같이 코드를 작성하였다. 이 코드를 작성하기 위해 사용하기 위한 프로그램은 ArcMap과 Python이며 이 두 프로그램을 연동시킬 수 있는 arcpy 모듈을 사용하였다. 이 코드는 모델을 구축하기 위해 ArcMap의 모델 빌더만을 활용하게 되면 경유지가 추가될 때마다 모델 자체를 계속해서 추가해야한다는 비효율적인 문제를 해결하기 위해 ArcMap의 각 기능들을 arcpy로 구현하고 for 구문을 적용한 것이다.

{% highlight ruby %}
def FindRoute():
  arcpy.CheckOutExtension("Network")
  from arcpy import env
  n = int(str(arcpy.GetCount_management("Converted_Graphics")))
  intermd = list()
{% endhighlight %}

{% highlight ruby %}
for i in range(1, n):
  env.workspace = "~/network.gdb"
  arcpy.Select_analysis("Converted_Graphics", "stop", "OID = %s OR OID =%s" % (i, i+1))
  env.overwriteOutput = True
  inNetworkDataset = "~/network.gdb/network_dataset/sidewalk"
  inStops = "~/network.gdb/stop"
  outGeodatabase = "~/network.gdb"
  outRoutes = "Routes"
  outRouteEdges = "RouteEdges"
  outDirections = "Directions"
  outStops = "Stops"
  measurement_units = "Minutes"

  arcpy.na.FindRoutes(inStops, measurement_units, inNetworkDataset, outGeodatabase, outRoutes,
  outRouteEdges, outDirections, outStops, Reorder_Stops_to_Find_Optimal_Routes=True,
  Preserve_Terminal_Stops="PRESERVE_BOTH")
  arcpy.CopyFeatures_management("Routes", "Sidewalk")
  arcpy.CalculateField_management("Sidewalk", "Name", "\"도보\"", "VB")

  inNetworkDataset = "~/network.gdb/network_dataset/bus_route"
  arcpy.na.FindRoutes(inStops, measurement_units, inNetworkDataset, outGeodatabase, outRoutes,
  outRouteEdges, outDirections, outStops, Reorder_Stops_to_Find_Optimal_Routes=True,
  Preserve_Terminal_Stops="PRESERVE_BOTH")
  arcpy.CopyFeatures_management("Routes", "Bus_route")
  arcpy.CalculateField_management("Bus_route", "Name", "\"버스\"", "VB")

  inNetworkDataset = "~/network.gdb/network_dataset/railroad"
  arcpy.na.FindRoutes(inStops, measurement_units, inNetworkDataset, outGeodatabase, outRoutes,
  outRouteEdges, outDirections, outStops, Reorder_Stops_to_Find_Optimal_Routes=True,
  Preserve_Terminal_Stops="PRESERVE_BOTH")
  arcpy.CopyFeatures_management("Routes", "Rail_road")
  arcpy.CalculateField_management("Rail_road", "Name", "\"지하철\"", "VB")

  # 출발 지점과 네트워크 경로와의 이격거리
  arcpy.Merge_management(["Sidewalk", "Bus_route", "Rail_road"], "Routes_merge")
  arcpy.Select_analysis("stop", "select1", "OBJECTID=1")
  arcpy.Near_analysis("Routes_merge", "select1", "", "NO_LOCATION", "NO_ANGLE", "PLANAR")
  arcpy.AddField_management("Routes_merge", "Total_meters", "DOUBLE", "", "", "", "", "NULLABLE",
  "NON_REQUIRED", "")
  arcpy.CalculateField_management("Routes_merge", "Total_meters", "[Shape_Length] + [NEAR_DIST]",
  "VB", "")

  # 도착 지점과 네트워크 경로와의 이격거리 및 총 소요시간 계산
  arcpy.Select_analysis("stop", "select2", "OBJECTID=2") 
  arcpy.Near_analysis("Routes_merge", "select2", "", "NO_LOCATION", "NO_ANGLE", "PLANAR")
  arcpy.CalculateField_management("Routes_merge", "Total_meters", "[Total_meters] + [NEAR_DIST]",
  "VB", "")
  arcpy.AddField_management("Routes_merge", "Total_Time", "DOUBLE", "", "", "", "", "NULLABLE",
  "NON_REQUIRED", "")
  arcpy.CalculateField_management("Routes_merge", "Total_Time",
  "( [Total_Minutes])+(( [Total_meters] - [Shape_Length]) /(4000/60) )", "VB", "")
  arcpy.Statistics_analysis("Routes_merge", "Route1_min", "Total_Time MIN", "")
  arcpy.JoinField_management("Routes_merge", "Total_Time", "Route1_min", "MIN_Total_Time", "")

  arcpy.Select_analysis("Routes_merge", "ROUTE%s" % i, "Total_Time = MIN_Total_Time")
  arcpy.DeleteField_management("ROUTE%s" % i, "StopCount;Total_Minutes;FirstStopOID;
  LastStopOID;Total_Kilometers;NEAR_FID;NEAR_DIST;FREQUENCY;MIN_Total_Time;Total_Miles")

  # 중간 결과물을 a 리스트에 저장
  intermd.append("ROUTE"+str(i))
  arcpy.Delete_management("Route%s" % (i))
  arcpy.Delete_management("Route%s" % (i+1))
{% endhighlight %}

{% highlight ruby %}
  #결과
  arcpy.Merge_management(intermd, "Final_ROUTE")
{% endhighlight %}