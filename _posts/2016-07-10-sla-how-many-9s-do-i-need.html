---
layout: post
title: 'SLA: How many 9''s do I need?'
date: 2016-07-10 21:37:07.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Random
tags: []
meta:
  _edit_last: '1'
  _jetpack_related_posts_cache: a:1:{s:32:"8f6677c9d6b0f903e98ad32ec61f8deb";a:2:{s:7:"expires";i:1525303673;s:7:"payload";a:0:{}}}
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<h2>Introduction</h2>
<p>I recently had a conversation with a colleague regarding service level agreements and what kind of up-time SLAs we were required to provide (or would recommend) to some our customers. This is something that comes up more and more, particularly in relation to software delivery on cloud hosting platforms. Azure, Amazon AWS, Open Stack, Rack Space, Google App Engine, and so on all offer ever increasing levels of improved up-time around their cloud offerings and this trickles down to the ISVs who build software on these platforms. So how many 9’s does your organisation’s system need ?</p>
<h2 id="percentage-availability">Percentage availability</h2>
<p>Availability is the ability for your users to access or use the system. If they can’t access it because it’s locked up, or offline, or the underlying hardware has failed, then it is unavailable.</p>
<p>For the uninitiated, measuring availability in 9’s is industry parlance for what percentage of time your application is available.  The following table maps out the equivalent allowed downtime described by those numbers.</p>
<table>
<thead>
<tr>
<th>Description</th>
<th>Up-time</th>
<th>Downtime per year</th>
<th>Downtime per month</th>
</tr>
</thead>
<tbody>
<tr>
<td>two 9’s</td>
<td>99%</td>
<td>~3.65 days</td>
<td>~7.2 hours</td>
</tr>
<tr>
<td>three 9’s</td>
<td>99.9%</td>
<td>~8.7 hours</td>
<td>~43 minutes</td>
</tr>
<tr>
<td>three and a half 9’s</td>
<td>99.95%</td>
<td>~4.3 hours</td>
<td>~21 minutes</td>
</tr>
<tr>
<td>four 9’s</td>
<td>99.99%</td>
<td>~52 minutes</td>
<td>~4.3 minutes</td>
</tr>
<tr>
<td>five 9’s</td>
<td>99.999%</td>
<td>~5.25 minutes</td>
<td>~25 seconds</td>
</tr>
</tbody>
</table>
<h2 id="service-level-agreements">Service Level Agreements</h2>
<p>How many 9’s a company or services’ SLA specifies, does not necessarily mean that the system will always adhere to or guarantee that level of up-time. No doubt, there are mission critical systems out there that would need guaranteed/consistent up-time and multiple layers of fail-over/redundancy in case those guarantees are not met. However, more often that not, these numbers are goals to be attained, and customers might be offered a rebate/credit if the availability did not reach those goals.</p>
<p>Take <a href="https://aws.amazon.com/s3/sla/" title="Amazon S3 SLA">Amazon S3 storage services</a> for example. Their service commitment goal is to maintain a three 9’s level of up-time in each month, however in the event that they do not, they offer a customer credit of: <br />
	- 10% in the case where they drop below three 9’s  <br />
- 25% in the case where they drop below two 9’s</p>
<p>Microsoft Azure has a similar service commitment for their <a href="https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_0/" title="SLA for Virtual Machines">IaaS Virtual Machines</a>. In this case, while they offer a similar credit rebate for dropping below, 99.95% they also caveat that you must have a a minimum of 2 virtual machines configured in an availability set across different fault domains (areas of their comm center infrastructure that ensure resources like power &amp; network are redundantly supplied). </p>
<h2 id="what-are-your-requirements">What are your requirements?</h2>
<p>Our business is predominantly focused on providing our customers with line of business applications. The large majority of their usage is by end-users between 8 am and 6 pm on business days. As a result, we have a level of flexibility with our customers to co-ordinate releases, planned outages and system maintenance in a way that minimally impacts the user base.</p>
<p>In the past however, I’ve built and maintained systems that were both financially and time critical; SMS based revenue generation based on 30 second TV ad spots for example have a very different business use case, requiring a different level of service availability. If you're system is offline during the 90 second window from the start of the advert, then you risk having lost that customer.</p>
<p>When identifying your own requirements, you need to think about the following:</p>
<ul>
<li>When do you need your system or application to be available?</li>
<li>Do you have different levels of availability requirements depending on time of day, month or year? 
<ul>
<li>LOB application that needs to be available 9-5/M-F</li>
<li>FinSrv application required for high availability at end of month but low availability through out the month</li>
<li>An e-commerce application requiring 24/7 availability across multiple geographic locations &amp; overlapping timezone</li>
</ul>
</li>
<li>What are the implications for your system being unavailable? 
<ul>
<li>Are there financial implications?</li>
<li>Is the usage/availability time critical/sensitive?</li>
<li>Are other systems upstream/downstream dependent upon you and if so, what SLA do they provide? </li>
</ul>
</li>
<li>If one component of your system is unavailable, is the entirety of the system unusable? 
<ul>
<li>Is component availability mutually exclusive?</li>
</ul>
</li>
</ul>
<h2 id="the-cost-of-higher-levels-of-availability">The cost of higher levels of availability</h2>
<p>Requiring higher levels of availability (more 9’s) means having a more complex, robust and resilient hardware infrastructure and software system. If your system is complicated, that may mean ensuring that the various constituent components can each, independently satisfy the SLA. e.g.</p>
<ul>
<li>Clustering your database in a Master-Master replication setup over multiple servers</li>
<li>Load-balancing your web application across multiple virtual machines</li>
<li>Redesigning to remove single points of failure in your application architecture such as in process session-state</li>
<li>Externalising certain services to 3rd parties that provide commercial solutions. (Azure Service Bus, Amazon S3 Storage etc…)</li>
</ul>
<p>And all these things comes with a cost. </p>
<h2 id="johns-e-commerce-site">Johns E-Commerce Site</h2>
<p>John runs an e-commerce website where he sells high value consumer goods. During the year his system generates ~€12m in revenue. Over the course of the year up-time equates to the following average revenue earnings, however since his business is low volume/high margin, missing a single sale/transaction could be costly.</p>
<ul>
<li>€1,000,000 per month</li>
<li>€33,333.33 per day</li>
<li>€1,388.89 per hour</li>
<li>€23.15 per minute</li>
</ul>
<p>John’s application currently only offers two 9’s of availability as it’s implemented on a single VPS and has numerous single points of failure. Planned outages are kept to a minimum but required to perform updates, releases and patches.</p>
<p>John is considering attempting to increase his platforms availability to four 9’s. Should he do it?</p>
<h2 id="quantifying-the-value-of-higher-levels-of-availability">Quantifying the value of higher levels of availability</h2>
<p>If you take a purely financial view of John’s situation, the cost implications of two 9’s vs. four 9’s is significant.</p>
<table>
<thead>
<tr>
<th>SLA</th>
<th>Outage Window</th>
<th>Formula</th>
<th>Total Cost of Max. Outages</th>
</tr>
</thead>
<tbody>
<tr>
<td>99%</td>
<td>3.65 Days</td>
<td>3.65 * €33333</td>
<td>€121,665.45</td>
</tr>
<tr>
<td>99.99%</td>
<td>52 minutes</td>
<td>52 * €23.15</td>
<td>€1,203.80</td>
</tr>
</tbody>
</table>
<p>Ultimately, he needs to understand if this is an accurate estimation of the cost impact, and if it is, would it cost him more than €120K year on year, to increase the up-time of his system. There are numerous other business and technical considerations here on both sides of the equation.</p>
<ul>
<li>Revenue estimation year on year may or may not be accurate</li>
<li>Revenue generation may not be evenly distributed through the year; if he can maintain high availability through the Black Friday and Christmas shopping seasons, it may alleviate most of his losses.</li>
<li>There may be other less tangible impacts on recurring revenue due to bad user experiences of arriving while the site is down etc.</li>
<li>Downtime may have a detrimental/negative impact on his brand.</li>
</ul>
<p>On the other hand, what is the cost of the upgrade.</p>
<ul>
<li>Development costs to upgrade the system.</li>
<li>Additional hosting costs to move to a cloud a platform or additional 3rd parties</li>
<li>On-going support costs to maintain this new system</li>
<li>There may be other considerations where the adoption of new technologies (a high availability cache) would alleviate the necessity of an increased SLA for a data store for example.</li>
</ul>
<p>Assuming that the system can be initially upgraded and maintained year on year for less than €120K, the return on investment would make sense for John to undertake this work. It would be a different conversation the next time though when he wants to go to five 9’s availability. </p>
<h2 id="thoughts">Thoughts?</h2>
<p>Deciding on an appropriate level for your SLA is complicated, and there are a myriad of considerations and inputs which will dictate the “right” answer for your particular situation. Whatever you decide, attempting to achieve higher and higher levels of availability for your system, will most probably lead to higher costs, and the smaller returns on investment. So make sure the level you choose is appropriate from both a business and technical perspective.</p>
<p><i>~Eoin Campbell</i></p>
