# Artifacts-detection-and-removal
file_path = 'rec_1m.mat';
load(file_path);
ecg_signal = val;
sampling_frequency = 500; % Hz
t = (0:numel(ecg_signal)-1) / sampling_frequency;
% Plot the original ECG signal
figure;
plot(t, ecg_signal);
title('Original ECG Signal');
xlabel('Time (s)');
ylabel('Amplitude (mV)');
grid on;
% Design a Butterworth bandpass filter
low_cutoff_frequency = 0.5; % Low cutoff frequency in Hz
high_cutoff_frequency = 50; % High cutoff frequency in Hz
filter_order = 4; % Filter order
% Normalize the cutoff frequencies
normalized_low_cutoff = low_cutoff_frequency / (sampling_frequency / 2);
normalized_high_cutoff = high_cutoff_frequency / (sampling_frequency / 2);
% Design the bandpass filter
[b, a] = butter(filter_order, [normalized_low_cutoff,
 normalized_high_cutoff], 'bandpass');
% Apply the bandpass filter to remove noise
ecg_signal_filtered = filtfilt(b, a, ecg_signal);
% Plot the original and filtered ECG signals
figure;
subplot(2, 1, 1);
plot(t, ecg_signal);
title('Original ECG Signal');
xlabel('Time (s)');
ylabel('Amplitude (mV)');
grid on;
subplot(2, 1, 2);
plot(t, ecg_signal_filtered);
title('Noise removal from ECG Signal');
xlabel('Time (s)');
ylabel('Amplitude (mV)');
grid on;
% Differentiate the filtered signal
ecg_signal_diff = diff(ecg_signal_filtered);
% Square the differentiated signal
ecg_signal_squared = ecg_signal_diff.^2;
% Integrate the squared signal
ecg_signal_integrated = cumsum(ecg_signal_squared);
% Apply a moving window for smoothing
window_size = round(sampling_frequency * 0.150); % 150 ms window
ecg_signal_smoothed = movmean(ecg_signal_integrated, window_size);
% Find peaks in the smoothed signal (QRS complex detection)
[~, qrs_indices] = findpeaks(ecg_signal_smoothed, 'MinPeakHeight', max(ecg_signal_smoothed) * 0.6);
% Identify R-peaks as the peaks of the QRS complexes
rpeak_indices = qrs_indices;
artifact_indices = find(ecg_signal > max(ecg_signal) * 0.8);
% Remove detected artifacts
ecg_signal_cleaned = ecg_signal;
ecg_signal_cleaned(artifact_indices) = [];
% Plot the original signal with QRS complexes, R-peaks, and identified artifacts
figure;
subplot(3, 1, 1);
plot((1:length(ecg_signal)) / sampling_frequency, ecg_signal);
hold on;
plot(qrs_indices / sampling_frequency, ecg_signal(qrs_indices), 'rx', 'MarkerSize', 8);
title('ECG Signal with QRS Complex Detection');
xlabel('Time (s)');
ylabel('Amplitude (mV)');
legend('ECG Signal', 'QRS Complexes');
grid on;
subplot(3, 1, 2);
plot((1:length(ecg_signal)) / sampling_frequency, ecg_signal);
hold on;
plot(artifact_indices / sampling_frequency, ecg_signal(artifact_indices), 'bx', 'MarkerSize', 8);
title('ECG Signal with Artifact Identification');
xlabel('Time (s)');
ylabel('Amplitude (mV)');
legend('ECG Signal', 'Detected Artifacts');
grid on;
subplot(3, 1, 3);
plot((1:length(ecg_signal_cleaned)) / sampling_frequency, ecg_signal_cleaned);
title('ECG Signal after Artifact Removal');
xlabel('Time (s)');
ylabel('Amplitude (mV)');
grid on;
