# EE109 Digital System Lab Final Report
Akshay Rajagopal, Andrea Ramirez
## Table of Contents
- Application Overview
- Software Simulation
- Hardware Implementation
- Design Tradeoffs
- Appendix
## Application Overview
```
Machine learning is a popular technique that allows us to train machines to carry out some task without having to program all the specifics. The machine learns how to tackle a certain task and, if well trained, is able to carry out this task with any input given. We wanted to explore one such machine learning algorithm and see how we could implement it on hardware, as well as analyze how the algorithm performed in hardware compared to software. We decided to analyze this hardware versus software comparison by trying to create a handwriting recognition program. More specifically, we built an application that trains on MNIST images, a database of 70,000 20x20 handwritten digits, and then classifies test images, also from the MNIST database, into the digit it perceives the image represents. 
We built two versions of this application in both software and hardware. One version uses 10 Support Vector Machines, or SVMs, that are trained on one digit, and then calculate the probability an image represents the digit it was trained to recognize (Although the output is not a probability in the true sense, it is essentially a measure of how likely a given input lies in that category; hence, we shall refer to it as a probability throughout this report). We then classify the image to be the digit with the highest probability, according to the SVMs. Our second version uses a neural network, a network with different layers of neurons that connect inputs to output classes, to classify images. The neurons in the neural network only activate for the stimuli they were trained for, which helps classify the images to the correct class. 
Overall, we found that the neural network worked the best. We were also able to get very good accuracies from our 4 versions (2 software, 2 hardware). We were also able to practice how to build a neural network, and honed our optimization skills, which led to hardware performance comparable to the software. In the next sections, we will discuss our software and hardware implementations, as well as the performance outcomes.
You can find all of our files and work in: https://github.com/akshay-rajagopal/ee109-project
```
## Software Simulation 
```
For our software implementation, we used MATLAB. We initially wrote our handwritten digit recognition application using SVMs and .mat files that held the information of the MNIST images. We loaded the data onto MATLAB and set different parameters used to minimize the function. We trained 10 SVMs, each trained to recognize a digit. The SVMs were trained using a stochastic subgradient method. We trained each SVM on 60,000 images. After all the training, we had a Weights matrix to use for testing, with each column of the matrix corresponding to one digit. We tested on 10,000 images. To classify images, we got a probability that an image represented each digit using the weights of each SVM. We then looked for which SVM gave the largest probability and chose that to classify the image. Finally, we compared our classification with the correct labels of each image to get the accuracy of our application. After running this simulation, we found our SVM classifier had 86.98% accuracy.  Our final software implementation is found at SVMTest.m
```
## Hardware Implementation
```
For our hardware implementation, we translated our SVM implementation to Spatial. We imported the images as csv files into Spatial, saving them to DRAM. We initially chose a Float type to do all our calculations. After initializing our data and setting the ports, we moved to the accelerator to do the training and testing. Just like in software, we trained 10 SVMs with each image by using Foreach controllers. Once we had our weights, we tested on other images using the same procedure as in software, getting the max probability from all the SVMs to classify an image to a digit. We also calculated the accuracy using the labels within the FPGA and sent this error back to the CPU to be displayed at the end of our code. Because Scala simulations, VCS, and synthesis take so long, we tested our implementation with a smaller set of images. Initially, we had 42 out of 100 errors after training on 600 images on Scala versus 34 errors on those same test and train condition on MATLAB. This showed us we had some optimization to do. 
After optimizing we changed our data type to Fixed Point with 5 pre and 27 post decimal places (FixPt[TRUE, 5, 27]). This allowed our code to compile with more ease and improved our accuracy, mainly because the weight values are small so we need more precision. It also improved speed. With this change, we were able to get to 34 errors out of 100 in hardware, matching our software implementation. We also added some parallelization factors and Reduce controllers to optimize, the effects of these are discussed in the Design Tradeoffs section.
Through our hardware testing phase, we became interested in comparing two hardware implementations. We were curious on how Neural Networks worked, which is why we also created a neural network application for our handwritten digit classifier. We used TensorFlow to calculate the weights and biases for all the layers in our network. These weights and biases were obtained using the training images. We then loaded these values as csv files into DRAM. We again used a fixed point representation for this implementation. After loading weights, parameters, and test images, we worked on FPGA testing. Our neural network had one hidden layer of 1024 neurons with ReLu activation and an output layer of 10 neurons, one per digit. For each image, we used the weights of our hidden layer and our final layer to get the probability of the image belonging to a digit. We then, once more, classified depending on the maximum probability, calculate the accuracy, and sent it back to the CPU for display. We ran this implementation on 200 test images and got 99.5% accuracy. A software implementation on the full dataset gave us 98.5% accuracy.  We also compared to a software implementation of neural networks we wrote and found they were the same. We discuss our thoughts on designing SVM versus Neural Network classifiers in the next section.  The tensorflow code is new_multilayer_perceptron.py and the Matlab inference engine implementation is NeuralNetTest.m
Please see system diagrams in img folder.
```
## Design Tradeoffs
```
Initially, we developed the classifiers for the Arria10 board.  Although VCS was successful here, synthesis could not complete because of timing constraints not being met.  Therefore, we created another version to synthesize on the ZCU board.  We first discuss our development with the Arria10 in mind, then briefly talk about the modifications made for ZCU.
After testing both of our implementations, we found that using neural networks to classify digits worked best. Our neural network ran for 71,099,856 cycles over 200 images, compared to the 4,767,400 cycles for the SVM over 600 training and 100 testing images. We added parallelization to both these implementations which helped increase the speed in exchange for a little more complexity. We largely parallelized the testing portion of the application, where we find the probabilities of an image corresponding to each digit. Because training had a lot more associated logic, this limited our ability to perform large-scale parallelization there, but we were able to introduce some parallelization there as well.  We also used reduce controllers, which helped parallelize our code further, and made our code run more smoothly. It added a little complexity, but it helped a lot in the end to make our design use its resources better. The parallelization helped us especially in synthesis, where resource utilization and default pipelining were both giving us issues. With some extra controllers and work, we were able to smooth out these errors. 
In terms of utilization, we found the SVM had 31% logic and 16% memory utilization, compared to 79% logic and 15% memory utilization for the neural network. This indicates to us that our neural net was easier to design in a way that would use resources most efficiently without running out them. The SVM on the other hand still had room for improvement, which would require more careful analysis of the code and more complexity to improve its efficiency. 
In order to fully synthesize the designs, we had to switch to the ZCU board.  This required us to change our parallelization a bit for the NN.  Here, our SVM used 67.5% logic and the neural net used 68.7% logic.  With all 10,000 images on board, the NN ran for 3,274,003,267 cycles, which translates to around 66 million cycles per 200 images.  
We also compared lines of code (LOC) on software versus hardware implementation. For the SVM, we had 47 software LOC versus 118 hardware LOC. In neural networks we had 17 software LOC versus 96 hardware LOC. This shows us that in general, neural network design is shorter compared to the SVM implementation, a fair tradeoff given the added complexity in understanding and creating a network. We also notice that the hardware versions have more LOC, which is mainly due to having to use many lines to initialize memory, ports, communication and other parts that hardware needs defined to function. 
We notice in general that neural networks seem to be the better design choice. For some added complexity, we get greater accuracy compared to SVMs. We also notice that both implementation worked better after parallelizing, as resources are better managed and we have less resources waiting instead of working. The Reduce controllers also helped with this parallelization, mainly in parallelizing matrix multiplication effectively. 
```
## Appendix
```
All the code, system diagrams, and csv files can be found at https://github.com/akshay-rajagopal/ee109-project.  The data required to run the Neural Net and SVM can be found in DataNN.zip and DataSVM.zip respectively.
After doing this final project, we were both able to practice our skills designing digital systems. It was a very nice experience that allowed us to work on something new that we were both interested in. We want to thank our teaching staff for helping us make this project a success and for giving us great ideas to take full advantage of the opportunity to work on digital design.
```
##Final Thoughts on EE109
```
Akshay
Overall I think EE109 was a nice class.  Although I had gotten a small taste of hardware-software co-optimization in EE180, I really enjoyed delving more into it in EE109 and applying to a bigger project.  As I had recently started taking classes in Convex Optimization and Machine Learning, I was excited to be able to work in that area for the project and really focus on optimizing frameworks through specialized hardware.  The higher-level abstraction of Spatial definitely helped us with building larger-scale projects in the few weeks we had than would have been possible with something like Verilog.  There are still a few areas where Spatial was a bit buggy, but fortunately Tian was able to help us with these issues.  Being able to see grades for the labs earlier in the quarter would have nice, but besides this and the Spatial bugs, I think the class was good.
```
```
Andrea
I really enjoyed being part of this course. I must admit it was a scary experience, this was the first class I went to where I was the only girl in the class, and that can be intimidating. I didnt think it would be as challenging as it was in terms of dealing with gender ratios, but I think it helped me become more confident as a woman in engineering. I would encourage though to try and get more women in the class, to get more women interested in digital design so that they arent discouraged not to take the class because there are only guys in it. I would also appeciate a bit more feedback early on, things like grades throughout the quarter help make adjustments to improve, and as of now I have no grades or feedback on any assignment so I cant improve on the way I was workking throghout the quarter. Thanks again for this nice class!
```