FROM ruby:2.2
RUN apt-get update -qq && apt-get install --no-install-recommends -y build-essential && \
    rm -rf /var/cache/apt/*

ENV APP_HOME /app
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

COPY  Gemfile* $APP_HOME/
RUN bundle install
COPY . $APP_HOME

ENV COMMENT_DATABASE_HOST=comment_db \
    COMMENT_DATABASE=comments

CMD ["puma"]
