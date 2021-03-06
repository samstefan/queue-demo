require 'securerandom'
require 'nsq'

require_relative 'marathon.rb'

# Default values if environment variables are not set.
DEFAULTS = {
  'MSS_QUEUE_ENDPOINT' => '127.0.0.1:4150',
  'MSS_TOPIC_NAME' => 'microscaling-demo',
  'MSS_CHANNEL_NAME' => 'microscaling-demo',
  'MSS_PRODUCER_SLEEP_MS' => '100',
  'MSS_CONSUMER_SLEEP_MS' => '300'
}

# Get environment variable if set or return the default.
def env_or_default(var_name)
  if ENV.has_key?(var_name)
    result = ENV[var_name]
  else
    result = DEFAULTS[var_name]
  end
end

# Sleep between accessing the queue.
def sleep_millis(var_name)
  sleep_ms = env_or_default(var_name)
  sleep_value = Integer(sleep_ms).to_f / 1000

  sleep(sleep_value)
end

desc 'Write messages to the queue.'
task :producer do |task|
  begin
    # Connect to NSQ.
    producer = Nsq::Producer.new(
      nsqd: env_or_default('MSS_QUEUE_ENDPOINT'),
      topic: env_or_default('MSS_TOPIC_NAME')
    )

    # Send messages in an infinite loop.
    loop do
      begin
        message = SecureRandom.uuid

        # Write a message to the queue.
        puts "Sending: #{message}"
        producer.write(message)

        # Sleep after writing each message.
        sleep_millis('MSS_PRODUCER_SLEEP_MS')
  
      rescue StandardError => e
        puts "LOOP ERROR: #{e.inspect}"
        puts e.backtrace
      end
    end

  rescue SystemExit, Interrupt
    puts 'Shutting down'
  rescue StandardError => e
    puts "ERROR: #{e.inspect}"
    puts e.backtrace
  ensure
    # Close the connection
    unless producer.nil?
      producer.terminate
      puts 'Connection closed'
    end
  end
end

desc 'Read messages from the queue.'
task :consumer do |task|
  begin
    # Connect to NSQ.
    consumer = Nsq::Consumer.new(
      nsqd: env_or_default('MSS_QUEUE_ENDPOINT'),
      topic: env_or_default('MSS_TOPIC_NAME'),
      channel: env_or_default('MSS_CHANNEL_NAME')
    )

    # Listen for messages in an infinite loop.
    loop do
      begin
        # Pop a message off the queue.
        msg = consumer.pop
        puts "Receiving: #{msg.body}"
        msg.finish

        # Sleep after reading each message.
        sleep_millis('MSS_CONSUMER_SLEEP_MS')

      rescue StandardError => e
        puts "LOOP ERROR: #{e.inspect}"
        puts e.backtrace
      end
    end

  rescue SystemExit, Interrupt
    puts 'Shutting down'
  rescue StandardError => e
    puts "ERROR: #{e.inspect}"
    puts e.backtrace
  ensure
    # Close the connection
    unless consumer.nil?
      consumer.terminate
      puts 'Connection closed'
    end
  end
end
