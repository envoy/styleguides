# JSONApi Resources Style Guide

## Table Of Contents

* [Resource File Layout](#layout)

## Layout

### WIP - This is basic layout style guide for creating new resources files ( app/resources/api/v3/* )

Basic layout is should be in the following order, note that the contents of each section should be alphabetized.

1. Resources Options ( immutable, pagination ... )
1. Attributes
1. Read Only Attributes
1. Has One Relations
1. Has Many Relations
1. Event Actions ( before_*, after_*, around_* )
1. Filter Declarations
1. Method Overrides
1. Custom Methods


Example Resource ( based on the FlowResource ):

```
module API
  module V3
    class ExampleResource < BaseResource
      # Resource Options
      pagination: none

      # Attributes
      attribute :additional_guests
      attribute :auto_sign_out_minutes
      attribute :auto_sign_out_at_midnight
      attribute :enabled
      attribute :id_check
      attribute :name
      attribute :position
      attribute :use_existing_photo

      # Read Only Attributes
      readonly :created_at
      readonly :updated_at

      # Has One Relations
      has_one :agreement_page, foreign_key_on: :related
      has_one :badge, foreign_key_on: :related
      has_one :location
      has_one :photo_page, foreign_key_on: :related
      has_one :sign_in_field_page, foreign_key_on: :related
      has_one :summary_page, foreign_key_on: :related

      # Has Many Relations
      has_many :lots_of_these
      has_many :these_too

      # Event Actions
      around_create :create_default_pages

      before_save :authorize_create, on: :create
      before_save :authorize_update, on: :update


      # Filters
      filter :location, apply: lambda { |records, values, _options|
        records.where("flows.location_id IN (?)", values)
      }

      # Method Overrides
      def self.creatable_fields(context)
        fields  = super(context)
        owner   = context[:current_resource_owner]
        company = owner.company || owner.location&.company

        if company.nil? || company.features.disabled?(:group_sign_in)
          fields -= [:additional_guests]
        end

        fields
      end

      def self.records(options = {})
        owner   = options[:context][:current_resource_owner]
        scope   = Flow.where(location: owner.locations).not_deleted
        company = owner.company || owner.location&.company

        if company.nil? || company.features.disabled?(:multiple_visitor_types)
          scope = scope.order(id: :asc).limit(1)
        else
          scope = scope.order(position: :asc)
        end

        scope
      end

      def self.updatable_fields(context)
        fields  = super(context)
        owner   = context[:current_resource_owner]
        company = owner.company || owner.location&.company

        if company.nil? || company.features.disabled?(:group_sign_in)
          fields -= [:additional_guests]
        end

        fields
      end

      # @override ApplicationResource#destroy
      def destroy
        return true if _model.deleted?

        _model.soft_delete
      end

      # Custom Methods
      protected

      def authorize_create
        unauthorized! unless LocationPolicy.new(context[:current_resource_owner], location&._model).update?
      end

      def authorize_update
        unauthorized! unless FlowPolicy.new(context[:current_resource_owner], _model).update?
      end

      def create_default_pages
        ActiveRecord::Base.transaction do
          yield && _model.create_default_pages!
        end
      end

      def unauthorized!
        raise Pundit::NotAuthorizedError, "not allowed to access this location"
      end
    end
  end
end
```