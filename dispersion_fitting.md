This block below is to prevent accidentally running the program by adding a running confirmation window.
```matlab
choice = menu('Continue?','Yes','No');
if choice==2 | choice==0
   return;
end
```
Use mphopen to open the COMSOL dispersion model file (.mph). This model will be used to evaluate dispersions. Control parameters (Modulus, etc.) can be optimized with the MATLAB optimization file.
```
close all; clear all;
global iter Target model freq
mphopen
```
Two-column array to input experimental/test dispersion data. The first column is the frequency (kHz). The second column is wave speed (m/s).
```
dat = [ 2,	7.651080172; 
        2.5,    7.518560174;  
        3,	7.8457104; 
        3.5,    8.028079523;	
        4,	8.450217292; 
        4.5,	8.663541523;	
        5,	8.540607346];
```
###Main part of the optimization
This is the main part of the optimization step. Here we assume there is only one optimization variable, which is the storage Young's modulus (Es) of the plate.
```matlab
tic
for i=1:1:length(dat) % loop through the test data
    close all
    freq = dat(i,1)*1000; %extract and convert test frequency to hz
    c_e = dat(i,2); %extract test wavespeed in m/s
    Target = [c_e]; %set test wave speed as target
    E_temp=[];
    for j=1:1:5 % for each frequency/wavespeed pair, simulate j times
        close all
        X1 = optimizableVariable(strcat('x1'), [200e3,280e3],'Type','real'); % x1 is E_s in COMSOL, optimization search range from 20e3 to 50e3 Pa1
        X = [X1];
        Mr = 1; %number of optimization variables
        iter_max = 10; %number of optimization loops (10)
        N_LHS = Mr*20; %initial points = 20
        A = lhsdesign(N_LHS,Mr);
        InitialX = [A(:,1)*200e3+80e3];% initial range, should be the same as X1, A from 0 to 1
        iter = 0;
        % optimization process
        BO = bayesopt(@COMSOL_eval,X,'Verbose',2,...
                'AcquisitionFunctionName','expected-improvement',...
            'MaxObjectiveEvaluations', iter_max,...
            'IsObjectiveDeterministic', 1, ...
            'InitialX', array2table(InitialX),...
            'UseParallel',false);
        %results
        [x y] = min(BO.ObjectiveTrace)
        opt= table2array(BO.XAtMinObjective);
        min_obj = BO.MinObjective;
        %optimized E_s
        E = opt(1)
        E_temp = [E_temp,E];
    end
    E_store = [E_store;E_temp];
end
toc
```
`tic`andtoc one line 1 and line 35 at as a stopwatch to show the runtime of this main code. 
There are two loops here. The outer loop goes through all the test dispersion data dat, row by row. In each iteration, we will load a pair of frequency and wave speed into freq and c_e. They need to be converted to units of Hz and m/s. The target of the whole optimziation is to match model predicted wave speed to the experimental wave speed data, so we set the c_e as the target. Optimization is done within the nested loop from line 8 to line 32. The optimization will be run by N times (5 in the example) to evaluate statistics (average, deviation, etc.) of the optimization target (Es). For each run, you will have a optimized Es stored in E_temp array (1xN). N can be set to 1 if necessary. All optimized E will be stored in E_store, which is a MxN array. M is the length of the test data.
for i=1:1:length(dat) %outer loop
    close all
    freq = dat(i,1)*1000; %extract and convert test frequency to hz
    c_e = dat(i,2); %extract test wavespeed in m/s
    Target = [c_e];
    E_temp=[];
    for j=1:1:5 %nested loop
        ...%optimization codes to obtain the optimized E
        E_temp = [E_temp,E]; %E is the optimized modulus
    end
    E_store = [E_store;E_temp]
end
In the core part of the optimization, from line 10 to line 30, we first assign X1 to be a optimization variable, which will be the Young's modulus in the COMSOL plate dispersion model. Setting its range, for example, from 200e3 to 280e3 kPa. It means the optimization code will only search for the 'best' E within this range.
X1 = optimizableVariable(strcat('x1'), [200e3,280e3],'Type','real');
Then we put X1 into the optimization variable array X. Since we only have one optimization variable here, so it's a special case. Mr is the number of optimization varible. iter_max is the number of optimization iterations you want to run, which should be adjusted based on your own judgement. N_LHS is the number of initial points for the optimization, 20 is just an example and should be adjusted based on your own judgement.
X = [X1];
Mr = 1; %number of optimization variables
iter_max = 10; %number of optimization loops (10)
N_LHS = Mr*20; %initial points = 20
We then use two lines below to set initial points for evaluation. Pay attention here, for InitialX, since A goes from 0 to 1, you want to make sure all initial points stay in the range from 200e3 to 280e3 as you previously defined.
A = lhsdesign(N_LHS,Mr);
InitialX = [A(:,1)*200e3+80e3];
The optimization is facilitated by bayesopt from MATLAB. Detailed settings should be referred from MATLAB documentation. The evaluation function is COMSOL_eval, which is a sub-program calling COMSOL for calculation. The optimization goal is to minimize the output of COMSOL_eval.
iter = 0;
% optimization process
BO = bayesopt(@COMSOL_eval,X,'Verbose',2,...
                                'AcquisitionFunctionName','expected-improvement',...
                                'MaxObjectiveEvaluations', iter_max,...
                                'IsObjectiveDeterministic', 1, ...
                                'InitialX', array2table(InitialX),...
                                'UseParallel',false);
After the optimization, we extract the optimized modulus with E=opt(1).
[x y] = min(BO.ObjectiveTrace)
opt= table2array(BO.XAtMinObjective);
min_obj = BO.MinObjective;
%optimized E_s
E = opt(1)
MATLAB calling COMSOL 
COMSOL_eval is a objective function involving dispersion evaluation from COMSOL. The key is to pass parameters (such as Young's modulus E) from MATLAB to COMSOL, pass dispersion results from COMSOL to MATLAB. 
% fitting wavespeed to COMSOL dispersion results
function obj = COMSOL_eval(X)
        global iter Target model freq
        iter = iter + 1; % iteration counter
        model.param.set('E', table2array(X(1,1))); % set Storage Young's modulus in COMSOL to E_s ('E_s',..
        model.study('std1').run(); % run COMSOL simulation
        % figure(5); pd = mphplot(model,'pg2'); % plot vertical displacement field (1D)
        % in COMSOL the dset2 is a dataset that contains frequenecy and wavespeed,
        % plotted in figure (5) in COMSOL
        f = real(mphglobal(model,'freq','dataset','dset2','outersolnum','all')); % extract frequency from COMSOL
        c = real(mphglobal(model,'freq*2*pi/k','dataset','dset2','outersolnum','all')); % extract wavespeed from COMSOL
        c_p = interp1(f(2:end),c(2:end),freq,'spline'); %interpolate the COMSOL dispersion results with a spline, and extract 
        res = [c_p];
        obj = sum(abs(res - Target));
end
 
E should be the parameter defined in COMSOL. In this case, it is the Young's modulus.  table2array(X(1,1)) is the parameter E defined from MATLAB optimization scheme. This line set E from COMSOL as table2array(X(1,1)) from MATLAB.
model.param.set('E', table2array(X(1,1)));
Once parameters are all passed to COMSOL, use this line to execute COMSOL. 'std1' is the name of the Eigenfrequency Study in COMSOL. If the name is changed in COMSOL, this should also be changed here. 
model.study('std1').run();
In COMSOL, results section, under Data Sets, there should be a parametric solutions # related to the dispersion data. Check its properties and the tag name in it. In the example, this tag is dset2. So you can use these two lines to extract frequency and wave speed of the dispersion from the this COMSOL run.  
f = real(mphglobal(model,'freq','dataset','dset2','outersolnum','all')); % extract frequency from COMSOL
c = real(mphglobal(model,'freq*2*pi/k','dataset','dset2','outersolnum','all'));
Now we use this line to first interpolate dispersion data into a continous curve. We skipped the first data point since sometime that value is zero. You can adjust this number case by case, the bottom line is not letting abnormal points to disrupt the disperison interpolation. The interpolation curve shape should be as expected, such as a A0 mode. 
Then wave speeds were evaluated at freq points, so that we can make direct comparison with experimental data.
c_p = interp1(f(2:end),c(2:end),freq,'spline');
Finally, we set COMSOL predicted wave speeds c_p as res, and create a objective function obj to evaluate the distance between res and Target. obj can be defined as other functions to describe the distance between two sets of wavespeeds. Or you can make some other functions as the obejective funciton.
res = [c_p];
obj = sum(abs(res - Target));
What if you have two or more control parameters?
Such as for viscoelastic material, instead of having a single Young's modulus E, you have storage modulus Es and loss modulus El.
In MATLAB, you need to define two separate variables for each.
X1 = optimizableVariable(strcat('x1'), [200e3,280e3],'Type','real'); %for storage
X2 = optimizableVariable(strcat('x2'), [20e3,80e3],'Type','real'); %for loss
Now that the dimension of the search space is 2, so you need to make following changes.
X = [X1, X2];
Mr = 2;
InitialX = [A(:,1)*200e3+80e3,A(:,2)*60e3+20e3];
Likewise, you will need to store Es and El separately.
In COMSOL_eval, you need to pass two parameters to COMSOL
model.param.set('El', table2array(X(1,1)));
model.param.set('Es', table2array(X(2,1))); %or table2array(X(1,2)) if failed
