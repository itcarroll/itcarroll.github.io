FROM ruby:2.7.1

COPY Gemfile ./

RUN bundle install && \
    rm Gemfile*
