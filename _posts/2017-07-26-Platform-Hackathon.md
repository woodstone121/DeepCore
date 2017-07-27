---
layout: post
title: Platform Hackathon!
author: Ben Vencill
desc: A team of Data Scientists apply Deep Learning to GBDX.
keywords: data science, geospatial, machine learning, education, travel, hackathon, deep learning, yolo
published: True
---

![Hackathon]({{ site.baseurl }}/assets/images/hackathon/opening_slide.jpg){: .width="40%"}

Once a quarter, the DigitalGlobe Platform Team hosts a 3-day hackathon for developers to crack the GBDX platform open to new ideas, innovations, and inventions. My team from the Herndon office set out July 18th to hack the summer edition of the platform challenge.

# The Team

The Herndon team was comprised of some heavy-hitting data scientists, eager to start swinging at the platform:

  **Andrew Jenkins** the team lead, geospatial big data expert

  **Mahmoud Lababidi** expert at traversing neural networks better than a traveling salesman

  **Alan Schoen** self proclaimed _data plumber_

  **Ben Vencill** aka _bentern_, a quick-learning all-around all-star



# The Goal

Our team had a specific goal in mind with our hackathon project. We wanted to integrate our deep-learning algorithms in to the GBDX platform. For those not in the know, GBDX, which stands for Geospatial Big Data, is DigitalGlobe's cloud-based platform that provides clients access to the vast expanses of DigitalGlobe imagery, and analytic solutions to complement the huge store of data. We wanted to deploy our algorithms into GBDX in order to leverage the value of such a huge store of amazing data.


![Cloud]({{ site.baseurl }}/assets/images/hackathon/cloudslide.jpg){: .width="40%"}
 

# Game Plan

Heading into the challenge, we had done our research on the GBDX platform and knew what it would take to deploy our model. The GBDX environment uses Docker to set up dependencies, Python tasking scripts for deployment, with some some integration steps in between. For our demo, we wanted to be able to display results against DigitalGlobe map tiles, so a small web stack was needed.

![Workflow]({{ site.baseurl }}/assets/images/hackathon/workflowslide.jpg){: .width="40%"}

We chose Postgres/PostGIS to ingest our model's geospatial output polygons and Leaflet/Flask to display our map and results against DigitialGlobe imagery.


## Day 1

Fresh from a day of travel, our team set out early Wednesday morning ready to hack. We arrived at DigitalGlobe Headquarters early so we had a chance to explore the beautiful Westminster campus. Situated a few miles from the Rocky Mountains, the building's open design offers a wonderful view of Colorado's natural wonder. After wandering and exploring the campus, we were ready to get to work.

The Hackathon kicked off with a room full of excited platform developers. The organizers announced they had bounties, in the form of cash prizes, for teams willing to complete hard tasks through their project ideas, such as working with imagery datasets or integrating newly developed software features. The rules were simple: A team needed a name, and an awesome idea. Everything else was left to the creativity of the developers! Also announced was the **tortilla challenge**, a tradition at DigitalGlobe hackathons. The coveted chili pepper trophy is awarded to the hacker who consumes the most tortillas throughout the course of the hackathon. Provided, of course, were breakfast burritos and plenty of lunchtime opportunities to score tortillas as well. _What could go wrong?_

I came up with **geoHACKINGdomination** for our name and the team liked it. With a name and a great idea, we were all set for hacking!

After chatting with some of the Westminster folks, bouncing ideas around, and getting to meet awesome developers like **Jeff Naus** and **Nate McIntyre**, we headed off to the _Geospatial Cave_ (one of many cleverly named nooks in the Westminster Building) to start hacking!

Day One work involved getting our models set up, prepping data sets, and laying the foundation for a successful push into GBDX. Our deep learning algorithms had several dependencies that needed to be built, including access to Nvidia's CUDA GPU library, TensorFlow for running the models, and GDAL for wrangling geospatial data. In order to contain all of these dependencies smoothly, GBDX asks for a Docker image to run the task inside. With some guidance from Jeff Naus, I started work on building the Docker image. This proved to be a bigger task than it looked like initially, as the libraries we needed were heavy-duty.

By the end of a long day, Mahmoud had his MantisShrimp segmentation model training on a dataset Alan had prepared, and I had YOLO training on the same set for object detection. Andrew and I had looked into the complications of setting up the Docker image, and we decided to give it a fresh look in the morning. With the models humming away and a set of goals whiteboarded on the cave walls, we retired for the night.

## Day 2

When we arrived at Westminster the next morning, I was surprised to see just how spicy the tortilla challenge was heating up to be! **Dave Grason** and **Bill Greer** (the reigning champ) were clear front-runners, with a lot of tortillas down already. We were also presented with some GBDX swag in the form of dainty GBDX tea cups for all our caffeine needs, which, during a hackathon, is a lot!

![GBDxMUG]({{ site.baseurl }}/assets/images/hackathon/gbdxmug.jpg){:width="15%"}

With coffee under our belts, we were ready continue hacking down our Docker problems, and began setting up for our demo. With help from Jeff Naus again , Mahmoud wrote a GBDX task for deployment, and started building our demo front-end. Alan and I worked on getting the output and database pipeline running smoothly. Andrew hammered away at Docker foo most of the day.

By the end of the afternoon, we had built a pipeline to run MantisShrimp, convert its output polygons back to geospatially referenced polygons, store the output in our database instance, and feed those polygons to a lightweight Flask/Leaflet front end web app. With the infastructure in place, all we needed to do was deploy to GBDX, which required our Docker image to be working.

It turns out working with Docker is more complicated when GPU capabilities are involved, especially when huge libraries like GDAL are also required. Docker was a nightmare for Andrew, but he fought through it valiantly and got the dependencies in order by the end of the day.

We were confident in our ability to put the remaining pieces together the next morning, so we took the rest of the evening to explore the beautiful foothills of the Rockies. We drove up to Flagstaff Mountain just outside of Boulder, and took a few mile trek around the crest. Despite some cloudy looking weather and the threat of rain, none came, and the trip was lovely. The view of the mountains in the distance was great! The hike and dinner in Boulder afterwards was a refreshing change of pace from all the hacking! 

![Mahmoud]({{ site.baseurl }}/assets/images/hackathon/mahmoud.jpg){:height="404px" width="320px"}
![team]({{ site.baseurl }}/assets/images/hackathon/team.jpg){:width="50%"}

## Day 3: Delivery
Mad props to Andrew for solving the Docker problem late Thursday night, allowing us to go full steam ahead into our demo. Mahmoud had left MantisShrimp training overnight as well, putting us in a nice position for the day. Andrew and Mahmoud sat down with some of the platform guys in the morning to jam through getting our GBDX task set up properly. Unfortunately this process was time consuming and there were a few kinks in the system, so we were unable to get the full task deployed before the 1 o'clock presentations. While Alan was putting the finishing touches on his geospatial referencing script and data pipeline, Mahmoud ran some tests (in between debuging the task deployment) of his MantisShrimp segmentation model. The results were awesome:

![mantis_naked]({{ site.baseurl }}/assets/images/hackathon/mantis1.png){:width="80%"}

If you look closely, each polygon here (they're small) is the MantisShrimp model's predicition of where a car is in the image. 

Laying the car polygon images on top of OpenStreetMaps reveals some insight into the prediction:

![mantis_osm]({{ site.baseurl }}/assets/images/hackathon/mantis_osm.png){:width="80%"}

Notice how most of the cars line up nicely with the roads in OpenStreetMap. Beautiful!

Unfortunately we didn't have enough time for our task to be deployed in GBDX, since the process takes a while, and we ran into several more roadblocks that couldn't be resolved in a single morning. With another day's time we would have been up and running smoothly in GBDX, but alas, time was a crunch for us down the stretch. 


We got these results compiled into a slideshow and presented our work to the rest of the hackers. There were a lot of other impressive undertakings presented, in particular I was impressed with the **Dem Drapers** team, who created a draper for satelitte imagery to lay on top of a dem file. Their demo was really cool to see!

After presentations, each participant got to vote on their favorite projects to determine the winners. Our team got 3rd place overall out of 11 teams, which is awesome! Also, I'll give a shoutout to Dave Grason for winning the tortilla challenge. Herndon represented well!


# Wrap-Up
We learned a lot as a team participating in the Platform Hackathon. Here are our main takeaways:

1. **Deep Learning Requirements Can Be Cumbersome**: We spent a large chunk of our time hacking away at a Docker configuration, trying to get GPU capabilities, CUDA installations, and GDAL to all work in tandem. Our Docker files were large and cumbersome, leading to several re-dos over the course of the week. Nvidia-Docker provided the nessesary GPU configurations, but required a new Docker build as well. It was smooth sailing once the Docker was built, but it was a large complicated beast to a team that was relatively inexperience working with the technology.

2. **GBDX is NOT Beginner Friendly**: The platform team was an indispensible resource to us that made our trip to Westminster worthwhile alone. Being able to consult the knowledge of Jeff Naus and other GBDX developers proved nessesary several times over. We were a group of seasoned developers (save myself) and couldn't navigate the tasking by ourselves. GBDX is a powerful tool that should be accessible to groups wanting to leverage DigitalGlobe's wealth of pixels, so I think there's a lot of work to be done making the platform more accessible to developers and non-developers alike.

3. **Deep Learning and GBDX Have Just Met**: And I hope they become the best of friends. We are among the first groups try to deploy a complete Deep Learning framework to GBDX, which has different needs than other algorithms that see action on the platform. Once a good pipeline gets established, I think Deep Learning applications can and will thrive on the rich resources that Geospatial Big Data provide, and GBDX can be the environment in which to do just that. We're the early pioneers exlporing a rich landscape with exciting potential to grow and prosper. 

4. **The Platform Team is Awesome**: We owe a huge thanks to the platform team at DigitalGlobe for hosting the hackathon! It was a great experience, especially getting to work closely with a lot of the developers who knew their stuff inside and out. 



