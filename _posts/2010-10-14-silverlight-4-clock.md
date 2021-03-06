---
layout: post
title: Silverlight 4 Clock
date: 2010-10-14 13:09:37.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- C#
- Coding
- Silverlight
tags:
- clock
- debug
- draw
- ellipse
- line
- runtime
- Silverlight
- Silverlight 4
- Visual Studio
- Visual Studio 2010
- VS2010
- xaml
meta:
  _edit_last: '1'
  _thumbnail_id: '251'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><a href="http://trycatch.me/silverlight-4-clock/microsoft-silverlight-logo/" rel="attachment wp-att-251"><img src="{{ site.baseurl }}/assets/microsoft-silverlight-logo-150x150.png" alt="Silverlight" title="Silverlight" width="150" height="150" class="size-thumbnail wp-image-251" /></a><br />
I was trying to get my Development Environment up &amp; running the other day with Silverlight 4. It turns out that the Silverlight debug runtime isn't actually part of the standard client, or the Silverlight 4 Tools for Visual Studio.</p>
<p>Thanks to this <a href="http://forums.silverlight.net/forums/p/188683/433499.aspx">thread</a> I discovered</p>
<p>The "Silverlight managed debugging package" is part of the developer runtime, not the SDK or Tools.  Make sure you have the latest version of the developer runtime installed (available at <a href="http://go.microsoft.com/fwlink/?LinkID=188039">http://go.microsoft.com/fwlink/?LinkID=188039</a></p>
<p>On the plus side I did throw together this nice pretty clock just to test everything out.<br />
There's probably a dozen ways to break this, but it covers the basic for autosizing the grid, limiting the dimensiosn with min height &amp; max height sections, drawing lines &amp; drawing ellipses &amp; a small bit of math.</p>
<p><center><br />
<iframe src="/Code/1_Clock/SlvrClock.LibTestPage.html" style="width:300px; height:233px;"></iframe><br />
</center></p>
<p><strong>XAML</strong></p>
```xml
<UserControl x:Class="SlvrClock.Lib.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d" d:DesignHeight="300" d:DesignWidth="400" MinHeight="100" MinWidth="100">
    <Grid x:Name="LayoutRoot" Background="Black">
        <Line x:Name="lh" Stroke="Green" StrokeThickness="15" StrokeStartLineCap="Round" StrokeEndLineCap="Triangle" />
        <Line x:Name="lm" Stroke="Blue" StrokeThickness="10" StrokeStartLineCap="Round" StrokeEndLineCap="Triangle" />
        <Line x:Name="ls" Stroke="Red" StrokeThickness="5" StrokeStartLineCap="Round" StrokeEndLineCap="Triangle" />
        <Ellipse x:Name="el" Stroke="Yellow" StrokeThickness="5" />
    </Grid>
</UserControl>
```
<p><strong>C#</strong></p>
```csharp
public partial class MainPage : UserControl
{
    private Storyboard timer = new Storyboard(); //timer
    private const double radian = Math.PI / 180; //radian
    private bool started = false;
    private double MaxHandLength { get; set; } //shorter length of width:height
    private Point CenterPoint { get; set; }
        
    public MainPage()
    {
        InitializeComponent();
        timer.Duration = TimeSpan.FromMilliseconds(50);
        timer.Completed += new EventHandler(Timer_Completed);
        timer.Begin();
    }

    protected void Timer_Completed(object sender, EventArgs e)
    {
        double aw2 = LayoutRoot.ActualWidth / 2, ah2 = LayoutRoot.ActualHeight / 2;
        if (!started || CenterPoint.X != aw2 || CenterPoint.Y != ah2)
        {
            //Reset the centerpoint & ratios on startup or if the window resizes.
            CenterPoint = new Point(aw2, ah2);
            MaxHandLength = Math.Min(CenterPoint.X, CenterPoint.Y);
            el.Height = el.Width = ((MaxHandLength - 10) * 2);
            lh.X1 = lm.X1 = ls.X1 = CenterPoint.X;
            lh.Y1 = lm.Y1 = ls.Y1 = CenterPoint.Y;
            started = true;
        }
        var now = System.DateTime.Now;
        //line hour - apply partial split for smoother transition and update more than onces per second.
        ChangeHand(lh, MaxHandLength - 80, 30 * (now.Hour + ((double)now.Minute) / 60));
        //line minute
        ChangeHand(lm, MaxHandLength - 60, 6 * (now.Minute + ((double)now.Second) / 60));
        //line second
        ChangeHand(ls, MaxHandLength - 40, 6 * (now.Second + ((double)now.Millisecond) / 1000));
        timer.Begin();
    }

    protected void ChangeHand(Line l, double r, double a) {
        //Calculate the point on the circumference based on Center, Radius & Angle.
        var i = a * radian;
        l.X2 = CenterPoint.X + r * Math.Sin(i);
        l.Y2 = CenterPoint.Y + r * -Math.Cos(i);
    }
}
```
***Eoin Campbell***