%% Reynolds Number Calculator - CU Aeronautical High-Performance Flight Testbed
% Developed by: Natsumi Kakuda
% Computes Re = rho*V*c/mu across the flight envelope (stall through cruise)
% using standard atmosphere properties, so airfoil polars can be pulled
% at the correct Re rather than a generic "low Re" guess. Allows exploration
% to estimate chord as well. 
%
% Toolbox requirement: atmosisa
%
% Formula: Re = (rho * V * c) / mu
%   rho = air density at altitude [kg/m^3]
%   V   = true airspeed [m/s]
%   c   = wing chord [m]
%   mu  = dynamic viscosity at temperature [Pa.s] (Sutherland's law)
 
clear; clc; close all;
 
%% ---------------- USER INPUTS --------------------
% Velocity
V_stall_obj_imp    = 35;  % [mph]
V_stall_thres_imp  = 45;  % [mph]
V_cruise_thres_imp = 125; % [mph TAS]
V_cruise_obj_imp   = 150; % [mph TAS]

V_stall_obj    = V_stall_obj_imp*(12500 / 27969);
V_stall_thres  = V_stall_thres_imp*(12500 / 27969);
V_cruise_thres = V_cruise_thres_imp*(12500 / 27969);
V_cruise_obj   = V_cruise_obj_imp*(12500 / 27969);

% Altitudes
alt_den = 8000 * (0.3048); % Low-speed density altitude [m]
alt_boulderair = 5288 * (0.3048); % Boulder Model Airfield elevation [m]
alt_obj = 15000 * (0.3048); % Objective cruise altitude [m]
alt_thres = 12000 * (0.3048); % Threshold cruise altitude [m]

% Altitude sweep: sea level to objective altitude in 500-ft increments
alt_range_ft = (0:500:16000).'; 
alt_range    = alt_range_ft * (0.3048); % [m]

% ---------------- CANDIDATE CHORDS --------------------
chord_range_imp = 6:1:24; % Candidate chords [in]
chord_range = chord_range_imp * 0.0254; % Candidate chords [m]
plot_chords_imp = [8 10 12 14 16];
plot_chords     = plot_chords_imp*0.0254;

% Placeholder only need to replace
Re_min = 2.0e5;

%% ---------------- Atmosphere function 'atmosisa' ----------------
[T, a, P, rho, nu, mu] = atmosisa(alt_range);

% Force outputs into column vectors
T   = T(:); % Temperature [K]
a   = a(:); % Speed of sound [m/s]
P   = P(:); % Pressure [Pa]
rho = rho(:); % Density [kg/m^3]
nu  = nu(:); % Kinematic viscosity [m^2/s]
mu  = mu(:); % Dynamic viscosity [Pa*s]

%% ---------------- RFP DESIGN CONDITIONS --------------------

case_name = ["Objective low speed"; "Threshold low speed"; "Threshold cruise"; "Objective cruise"];

case_altitude_ft = [8000;8000;12000;15000];
case_altitude_m  = case_altitude_ft* 0.3048;
case_speed_mph   = [V_stall_obj_imp;V_stall_thres_imp; V_cruise_thres_imp;V_cruise_obj_imp];
case_speed_mps   = case_speed_mph*(12500 / 27969);

[case_T,case_a,case_P,case_rho,case_nu,case_mu] = atmosisa(case_altitude_m);

case_T = case_T(:); case_a = case_a(:); case_P = case_P(:);
case_rho = case_rho(:); case_nu = case_nu(:); case_mu = case_mu(:);

% Rows are RFP cases; columns are candidate chords
Re_cases = (case_rho.*case_speed_mps./case_mu)*chord_range;
Mach_cases = case_speed_mps./case_a;

Re_8000_obj    = Re_cases(1,:);
Re_8000_thres  = Re_cases(2,:);
Re_12000_thres = Re_cases(3,:);
Re_15000_obj   = Re_cases(4,:);

%% ---------------- MINIMUM CHORD CALCULATION --------------------

minimum_chord_m  = Re_min.*case_mu./(case_rho.*case_speed_mps);
minimum_chord_in = minimum_chord_m/0.0254;

[controlling_chord_in,controlling_case_index] = max(minimum_chord_in);

% Rounding upward to the nearest 0.5 inch
selected_chord_in = ceil(controlling_chord_in*2)/2;
selected_chord_m  = selected_chord_in*0.0254;
Re_selected = case_rho.*case_speed_mps.*selected_chord_m./case_mu;

fprintf('\nREYNOLDS-NUMBER CHORD SCREENING\n');
fprintf('--------------------------------\n');
fprintf('Selected Re_min: %.0f\n',Re_min);
fprintf('Controlling case: %s\n',case_name(controlling_case_index));
fprintf('Calculated minimum chord: %.2f in\n',controlling_chord_in);
fprintf('Chord rounded upward: %.1f in\n\n',selected_chord_in);

results = table(case_name,case_altitude_ft,case_speed_mph,case_rho,case_mu,Mach_cases,minimum_chord_in,Re_selected,'VariableNames',{'Case','Altitude_ft','Speed_mph_TAS','Density_kg_m3','DynamicViscosity_Pa_s','Mach','MinimumChord_in','Re_at_selected_chord'});
disp(results);

Re_table = table(chord_range_imp.',Re_8000_obj.',Re_8000_thres.', Re_12000_thres.',Re_15000_obj.','VariableNames',{'Chord_in','Re_35mph_8000ft','Re_45mph_8000ft','Re_125mph_12000ft','Re_150mph_15000ft'});

%% ---------------- REYNOLDS NUMBER CALC --------------------

% Rows correspond to altitude
% Columns correspond to candidate chord
Re_stall_obj = zeros(length(alt_range), length(chord_range));

for i = 1:length(alt_range)
    for j = 1:length(chord_range)
        Re_stall_obj(i,j) = rho(i) * V_stall_obj * chord_range(j) / mu(i);
    end
end

Re_stall_obj(i,j) = V_stall_obj * chord_range(j) / nu(i);

%% ---------------- PLOT 1: ATMOSPHERIC Properties --------------------

figure(1);
tiledlayout(2,2);
nexttile
plot(rho, alt_range_ft, 'LineWidth', 2);
grid on;
xlabel('Density [kg/m^3]');
ylabel('Altitude [ft]');
title('Air Density');

nexttile
plot(mu, alt_range_ft, 'LineWidth', 2);
grid on;
xlabel('Dynamic Viscosity [Pa s]');
ylabel('Altitude [ft]');
title('Dynamic Viscosity');

nexttile
plot(T, alt_range_ft, 'LineWidth', 2);
grid on;
xlabel('Temperature [K]');
ylabel('Altitude [ft]');
title('Temperature');

nexttile
plot(a, alt_range_ft, 'LineWidth', 2);
grid on;
xlabel('Speed of Sound [m/s]');
ylabel('Altitude [ft]');
title('Speed of Sound');

sgtitle('Standard Atmospheric Properties');



%% ---------------- PLOT 2: REYNOLDS NUMBER VS CHORD --------------------

figure(2);
hold on;
grid on;
box on;
plot(chord_range_imp,Re_8000_obj,'LineWidth',2.5,'DisplayName','35 mph at 8,000 ft');
plot(chord_range_imp,Re_8000_thres,'LineWidth',2.5,'DisplayName','45 mph at 8,000 ft');
plot(chord_range_imp,Re_12000_thres,'LineWidth',2.5,'DisplayName','125 mph at 12,000 ft');
plot(chord_range_imp,Re_15000_obj,'LineWidth',2.5,'DisplayName','150 mph at 15,000 ft');
yline(Re_min,'k--',sprintf('Re_{min} = %.0f',Re_min),'LineWidth',2,'HandleVisibility','off');
xline(selected_chord_in,'m--',sprintf('Selected chord = %.1f in',selected_chord_in),'LineWidth',2,'HandleVisibility','off');
plot(selected_chord_in,Re_selected(controlling_case_index),'ko','MarkerFaceColor','y','MarkerSize',9,'HandleVisibility','off');
xlabel('Chord [in]'); ylabel('Reynolds number');
title('Reynolds Number vs. Chord at RFP Design Conditions');
legend('Location','eastoutside');
hold off;

%% ---------------- PLOT 3: CHORD-ALTITUDE CONTOUR --------------------

Re_map = (rho.*V_stall_obj./mu)*chord_range;

figure(3);
contourf(chord_range_imp,alt_range_ft,Re_map,24,'LineColor','none');
hold on;
colorbar;
colormap(turbo);
[C,h] = contour(chord_range_imp,alt_range_ft,Re_map,[Re_min Re_min],'k','LineWidth',3);
clabel(C,h,'Color','k','FontWeight','bold');
yline(8000,'w--','8,000 ft','LineWidth',2);
yline(12000,'w--','12,000 ft','LineWidth',1.5);
yline(15000,'w--','15,000 ft','LineWidth',1.5);
xline(selected_chord_in,'m--','Selected chord','LineWidth',2.5, 'FontSize', 12, 'FontWeight','bold');
xlabel('Chord [in]'); ylabel('Altitude [ft]');
title('Reynolds Number at 35 mph');


%% ---------------- PLOT 4: REYNOLDS NUMBER VS ALTITUDE --------------------

Re_altitude = (rho.*V_stall_obj./mu)*plot_chords;

figure(4);
hold on;
grid on;
box on;
plot(alt_range_ft,Re_altitude,'LineWidth',2);
yline(Re_min,'k--',sprintf('Re_{min} = %.0f',Re_min),'LineWidth',2,'HandleVisibility','off');
xline(8000,'--','8,000 ft','HandleVisibility','off');
xline(12000,'--','12,000 ft','HandleVisibility','off');
xline(15000,'--','15,000 ft','HandleVisibility','off');
xlabel('Altitude [ft]'); ylabel('Reynolds number');
title('Reynolds Number vs. Altitude at 35 mph');
legend(compose('Chord = %.0f in',plot_chords_imp),'Location','southwest');
hold off;

%% ---------------- PLOT 6: REYNOLDS NUMBER VS AIRSPEED --------------------

speed_range_mph = (25:5:160).';
speed_range_mps = speed_range_mph*(12500 / 27969);

rho_8000 = case_rho(1);
mu_8000  = case_mu(1);
Re_speed = (rho_8000.*speed_range_mps./mu_8000)*plot_chords;

figure(6);
hold on;
grid on;
box on;
plot(speed_range_mph,Re_speed,'LineWidth',2);
yline(Re_min,'k--',sprintf('Re_{min} = %.0f',Re_min),'LineWidth',2,'HandleVisibility','off');
xline(35,'--','35 mph','HandleVisibility','off');
xline(45,'--','45 mph','HandleVisibility','off');
xlabel('True airspeed [mph]');
ylabel('Reynolds number');
title('Reynolds Number vs. Airspeed at 8,000 ft');
legend(compose('Chord = %.0f in',plot_chords_imp),'Location','eastoutside');
hold off;

%% ---------------- PLOT 7: RFP CASE HEATMAP --------------------

figure(7);
imagesc(chord_range_imp,1:numel(case_name),Re_cases);
set(gca,'YDir','normal');
colorbar;
colormap(turbo);
yticks(1:numel(case_name));
yticklabels(case_name);
xlabel('Chord [in]');
ylabel('RFP design condition');
title('Reynolds Number Across RFP Conditions');
hold on;
xline(selected_chord_in,'w--','Selected chord','LineWidth',2);
hold off;

%% ---------------- SAVE RESULTS commented out --------------------
% 
% writetable(results,'reynolds_mission_results.csv');
% writetable(Re_table,'reynolds_chord_results.csv');
